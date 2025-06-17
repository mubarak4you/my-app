module "common_data" {
  source = "./modules/common-data"

  environment                  = var.environment
  region_key                   = var.region_key
  short_name                   = var.short_name
  user_id                      = var.user_id
  vsad                         = var.vsad
  enable_whois_service_request = true
  enable_vast_service_request  = true
}

locals {

  # Cluster auth
  kubeconfig             = yamldecode(data.oci_containerengine_cluster_kube_config.cluster.content)
  kubeconfig_clusters    = lookup(local.kubeconfig, "clusters", [])
  cluster_ca_certificate = base64decode(local.kubeconfig_clusters[0]["cluster"]["certificate-authority-data"])

  # Node pool configuration
  availability_domain1   = data.oci_identity_availability_domain.ad[0].name
  availability_domain2   = data.oci_identity_availability_domain.ad[1].name
  worker_node_pool1_name = "workload-${lower(substr(local.availability_domain1, 5, -1))}"
  worker_node_pool2_name = "workload-${lower(substr(local.availability_domain2, 5, -1))}"
  worker_node_pool1_id   = oci_containerengine_node_pool.node_pool[local.worker_node_pool1_name].id
  worker_node_pool2_id   = oci_containerengine_node_pool.node_pool[local.worker_node_pool2_name].id
  min_max_nodes          = "${var.min_nodes_per_node_pool}:${var.max_nodes_per_node_pool}"
  nodes = join(", ", [
    "${local.min_max_nodes}:${local.worker_node_pool1_id}",
    "${local.min_max_nodes}:${local.worker_node_pool2_id}"]
  )
  node_pools = {
    (local.worker_node_pool1_name) = {
      availability_domain = local.availability_domain1
    }
    (local.worker_node_pool2_name) = {
      availability_domain = local.availability_domain2
    }
  }

  # Load balancer
  cluster_hostname = replace(module.common_data.cluster_name, ".", "-")

  # Get OKE Linux image for kubernetes version
  k8s_version_major = regex("v([0-9]+)\\.[0-9]+\\.[0-9]+", module.common_data.cluster_config.kubernetes_version)[0]
  k8s_version_minor = regex("v[0-9]+\\.([0-9]+)\\.[0-9]+", module.common_data.cluster_config.kubernetes_version)[0]
  k8s_image_version = "${local.k8s_version_major}${local.k8s_version_minor}"
  k8s_image_regex   = "vz-okel8-xu-[0-9]{4}-[0-9]{2}-${local.k8s_image_version}-gold"
  oke_linux_images = [
    for image in data.oci_core_images.engineering.images : image
    if can(regex(local.k8s_image_regex, image.display_name))
  ]
  node_image_id = var.node_image_id != null ? var.node_image_id : local.oke_linux_images[0].id
}

data "oci_containerengine_cluster_kube_config" "cluster" {
  cluster_id = oci_containerengine_cluster.cluster.id
}

data "oci_core_images" "engineering" {
  compartment_id = module.common_data.environment_config.engineering_compartment_id
  sort_by        = "TIMECREATED"
  sort_order     = "DESC"
}

data "oci_identity_availability_domain" "ad" {
  count          = 2
  compartment_id = module.common_data.compartment_id
  ad_number      = count.index + 1
}

resource "oci_containerengine_cluster" "cluster" {
  compartment_id     = module.common_data.compartment_id
  kubernetes_version = module.common_data.cluster_config.kubernetes_version
  name               = module.common_data.cluster_name
  vcn_id             = module.common_data.network_config.vcn_id
  type               = var.cluster_type == "basic" ? "BASIC_CLUSTER" : "ENHANCED_CLUSTER"
  # We are limited to 10 freeform tags
  freeform_tags = merge(
    module.common_data.tags.cluster.freeform,
    {
      "ComputeOS"          = "OKE"
      "CreatedAt"          = formatdate("EEEE, DD-MMM-YYYY hh:mm:ss ZZZ", time_static.cluster_creation.rfc3339)
      "CreatedPipelineUrl" = terraform_data.pipeline_metadata.output.created_pipeline_url
    },
  )

  kms_key_id = module.common_data.kms_key_id

  endpoint_config {
    is_public_ip_enabled = false
    subnet_id            = module.common_data.network_config.endpoint_subnet_id
    nsg_ids              = module.common_data.network_config.endpoint_nsgs
  }
  cluster_pod_network_options {
    cni_type = var.cni_type == "flannel" ? "FLANNEL_OVERLAY" : "OCI_VCN_IP_NATIVE"
  }
  options {
    kubernetes_network_config {
      pods_cidr     = var.pods_cidr
      services_cidr = var.services_cidr
    }
    persistent_volume_config {
      defined_tags  = module.common_data.tags.block_storage.defined
      freeform_tags = module.common_data.tags.block_storage.freeform
    }
    service_lb_config {
      defined_tags  = module.common_data.tags.load_balancer.defined
      freeform_tags = module.common_data.tags.load_balancer.freeform
    }
    service_lb_subnet_ids = module.common_data.network_config.oci_lb_subnet_ids
  }

  provisioner "local-exec" {
    # Configure cluster with options not availabe to the TF resource
    environment = {
      "DISABLE_COREDNS_ADDON" = var.is_coredns_addon_enabled ? "FALSE" : "TRUE"
      "CLUSTER_ID"            = self.id
      "REGION_ID"             = var.region_key == "iad" ? "us-ashburn-1" : "us-phoenix-1"
    }
    working_dir = "${path.module}/scripts"
    interpreter = ["/bin/bash", "-c"]
    command     = "./init_cluster.sh"
  }
  timeouts {
    create = "2h"
    update = "3h"
    delete = "2h"
  }
}

resource "oci_logging_log_group" "oke_log_group" {
  compartment_id = module.common_data.compartment_id
  display_name   = module.common_data.cluster_name
  description    = "Control plane logs for ${module.common_data.cluster_name} cluster"
  freeform_tags  = module.common_data.tags.logging.freeform
}

resource "oci_logging_log" "oke_controlplane_logs" {
  display_name = module.common_data.cluster_name
  log_group_id = oci_logging_log_group.oke_log_group.id
  log_type     = "SERVICE"

  configuration {
    source {
      category    = "all-service-logs"
      resource    = oci_containerengine_cluster.cluster.id
      service     = "oke-k8s-cp-prod"
      source_type = "OCISERVICE"
    }

    compartment_id = module.common_data.compartment_id
  }

  freeform_tags      = module.common_data.tags.logging.freeform
  is_enabled         = true
  retention_duration = 30
}

resource "oci_containerengine_node_pool" "node_pool" {
  for_each = local.node_pools

  cluster_id     = oci_containerengine_cluster.cluster.id
  compartment_id = module.common_data.compartment_id
  name           = each.key
  node_shape     = var.node_shape
  freeform_tags = merge(
    module.common_data.tags.node_pool.freeform,
    { "ClusterID" = oci_containerengine_cluster.cluster.id }
  )
  defined_tags       = module.common_data.tags.node_pool.defined
  kubernetes_version = module.common_data.cluster_config.kubernetes_version

  node_config_details {
    is_pv_encryption_in_transit_enabled = true
    kms_key_id                          = module.common_data.kms_key_id
    nsg_ids                             = module.common_data.network_config.node_nsgs
    placement_configs {
      availability_domain = each.value.availability_domain
      subnet_id           = module.common_data.network_config.node_subnet_id
    }
    size = var.min_nodes_per_node_pool

    dynamic "node_pool_pod_network_option_details" {
      for_each = var.cni_type == "flannel" ? [1] : []
      content { # Flannel requires cni type only
        cni_type = "FLANNEL_OVERLAY"
      }
    }

    dynamic "node_pool_pod_network_option_details" {
      for_each = var.cni_type == "npn" ? [1] : []
      content { # VCN-Native requires max pods/node, nsg ids, subnet ids
        cni_type          = "OCI_VCN_IP_NATIVE"
        max_pods_per_node = min(max(var.max_pods_per_node, 1), 110)

        pod_subnet_ids = var.node_pod_subnet_ids
      }
    }

    defined_tags = merge(
      module.common_data.tags.node.defined,
      { "Compute-Tag.OS" = "OKE" }
    )
    freeform_tags = merge(
      module.common_data.tags.node.freeform,
      { "ClusterID" = oci_containerengine_cluster.cluster.id }
    )
  }

  node_eviction_node_pool_settings {
    eviction_grace_duration              = "PT1M"
    is_force_delete_after_grace_duration = true
  }

  node_pool_cycling_details {
    is_node_cycling_enabled = true
    maximum_surge           = "100%"
    maximum_unavailable     = "0%"
  }
  node_shape_config {
    memory_in_gbs = var.memory_in_gbs
    ocpus         = var.ocpus
  }
  node_source_details {
    image_id    = local.node_image_id
    source_type = "IMAGE"
  }

  node_metadata = {
    user_data = base64encode(
      templatefile(
        "${path.module}/scripts/oke_node_user_data.tftpl",
        {
          http_proxy = module.common_data.proxy.http_proxy
          no_proxy   = module.common_data.proxy.no_proxy
        }
      )
    )
  }
  lifecycle {
    ignore_changes = [
      # cappni7 - Ignoring changes to the node tags. We found that changing
      # the node tag failed due to this error:
      #  409-Conflict, Cannot perform nodepool cycling and nodepool Placement Configuration change simultaneously.
      # This limitation was raised with Oracle on 8/2/2024.
      node_config_details[0].defined_tags,
      node_config_details[0].freeform_tags,
      node_config_details[0].size,
    ]
  }
  timeouts {
    create = "2h"
    update = "3h"
    delete = "2h"
  }
}

resource "oci_containerengine_addon" "cluster_autoscaler" {
  addon_name                       = "ClusterAutoscaler"
  cluster_id                       = oci_containerengine_cluster.cluster.id
  remove_addon_resources_on_delete = false

  # add-on configuration arguments
  # https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengconfiguringclusteraddons-configurationarguments.htm#contengconfiguringclusteraddons-configurationarguments_ClusterAutoscaler
  configurations {
    key   = "authType"
    value = var.cluster_autoscaler_authtype
  }
  configurations {
    key   = "balanceSimilarNodeGroups"
    value = "true"
  }
  configurations {
    key = "balancingLabel"
    value = join(", ", [
      "kubernetes.io/arch",
      "kubernetes.io/os",
      "topology.kubernetes.io/region",
    ])
  }
  configurations {
    key   = "enforceNodeGroupMinSize"
    value = "true"
  }
  configurations {
    key   = "maxNodeProvisionTime"
    value = "25m"
  }
  configurations {
    key   = "maxNodesTotal"
    value = var.max_nodes_per_node_pool * length(local.node_pools)
  }
  configurations {
    key   = "nodes"
    value = local.nodes
  }
  configurations {
    key   = "numOfReplicas"
    value = "2"
  }
  configurations {
    key   = "skipNodesWithCustomControllerPods"
    value = "false"
  }
  configurations {
    key   = "skipNodesWithLocalStorage"
    value = "false"
  }
  configurations {
    key   = "skipNodesWithSystemPods"
    value = "false"
  }

  # https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#how-can-i-increase-the-information-that-the-ca-is-logging
  configurations {
    key   = "v"
    value = tostring(var.cluster_autoscaler_log_verbosity)
  }

  configurations {
    key   = "topologySpreadConstraints"
    value = "[{\"maxSkew\": 1, \"topologyKey\": \"topology.kubernetes.io/zone\", \"whenUnsatisfiable\": \"DoNotSchedule\", \"labelSelector\": {\"matchLabels\": {\"app\": \"cluster-autoscaler\"}}}]"
  }

  configurations {
    key   = "cluster-autoscaler.ContainerResources"
    value = "{\"limits\": {\"cpu\": \"500m\", \"memory\": \"200Mi\" }, \"requests\": {\"cpu\": \"100m\", \"memory\": \"100Mi\"}}"
  }
}


resource "oci_containerengine_addon" "certificate_manager" {
  addon_name                       = "CertManager"
  cluster_id                       = oci_containerengine_cluster.cluster.id
  remove_addon_resources_on_delete = false

  # add-on configuration arguments
  # https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengconfiguringclusteraddons-configurationarguments.htm#contengconfiguringclusteraddons-configurationarguments_CertificateManager
  configurations {
    key   = "numOfReplicas"
    value = "2"
  }
}

resource "terraform_data" "pre_delete_cluster" {
  input = {
    oci_cert_names = join(",", [
      "ic-prometheus-${local.cluster_hostname}",
      "ic-grafana-${local.cluster_hostname}"
    ])
    compartment_id = module.common_data.compartment_id
    region_id      = module.common_data.region_id
  }
  provisioner "local-exec" {
    when = destroy
    environment = {
      "OCI_CERT_NAMES"       = self.output.oci_cert_names
      "OCI_COMPARTMENT_OCID" = self.output.compartment_id
      "OCI_REGION"           = self.output.region_id
    }
    working_dir = "${path.module}/scripts"
    interpreter = ["/bin/bash", "-c"]
    command     = "python pre_delete_cluster.py"
  }

  depends_on = [oci_containerengine_cluster.cluster]
}

resource "time_static" "cluster_creation" {
}

resource "terraform_data" "pipeline_metadata" {
  input = {
    created_pipeline_url = module.common_data.pipeline_metadata.pipeline_url
  }

  lifecycle {
    ignore_changes = [
      input.created_pipeline_url,
    ]
  }
}



[alabimu@63-22-141-104 ~]$ for ns in $(kubectl get helmrelease --all-namespaces -o jsonpath='{.items[*].metadata.namespace}' | tr ' ' '\n' | sort -u); do
>   for name in $(kubectl get helmrelease -n "$ns" -o jsonpath='{.items[*].metadata.name}'); do
>     echo "Checking release: $ns/$name"
>     helm get values "$name" -n "$ns" 2>/dev/null | grep ETOKE-438 && echo "↳ Found in $ns/$name"
>   done
> done


Checking release: kube-system/coredns

Checking release: kube-system/fluentd
clusterName: oke.ETOKE-438.np.iad.go0v
  value: oke.ETOKE-438.np.iad.go0v
↳ Found in kube-system/fluentd

Checking release: kube-system/gatekeeper

Checking release: kube-system/gatekeeper-constraints-security
clusterName: oke.ETOKE-438.np.iad.go0v
↳ Found in kube-system/gatekeeper-constraints-security

Checking release: kube-system/gatekeeper-templates-security

Checking release: kube-system/gitlab-runner
  tags: oke.ETOKE-438.np.iad.go0v
↳ Found in kube-system/gitlab-runner


Checking release: kube-system/gitlab-runner-executor
Checking release: kube-system/grafana
Checking release: kube-system/kube-state-metrics
Checking release: kube-system/metrics-server
Checking release: kube-system/node-local-dns
Checking release: kube-system/oci-native-ingress-controller

Checking release: kube-system/oke-platform-ingress
    host: grafana-oke-ETOKE-438-np-iad-go0v.ebiz.verizon.com
      secretName: grafana-oke-ETOKE-438-np-iad-go0v
    host: prometheus-oke-ETOKE-438-np-iad-go0v.ebiz.verizon.com
      secretName: prometheus-oke-ETOKE-438-np-iad-go0v
        "VSAD": "GO0V", "Zone": "GZ"}, "OKE-tags": {"ClusterName": "oke.ETOKE-438.np.iad.go0v"}}'
      oci-native-ingress.oraclecloud.com/freeform-tags: '{"ClusterName": "oke.ETOKE-438.np.iad.go0v"}'
    name: grafana-oke-ETOKE-438-np-iad-go0v
        "VSAD": "GO0V", "Zone": "GZ"}, "OKE-tags": {"ClusterName": "oke.ETOKE-438.np.iad.go0v"}}'
      oci-native-ingress.oraclecloud.com/freeform-tags: '{"ClusterName": "oke.ETOKE-438.np.iad.go0v"}'
    name: prometheus-oke-ETOKE-438-np-iad-go0v
↳ Found in kube-system/oke-platform-ingress

Checking release: kube-system/oke-rbac-config
Checking release: kube-system/prometheus
Checking release: kube-system/prometheus-k8s-events-exporter
Checking release: kube-system/rbac-manager
Checking release: kube-system/reflector
Checking release: kube-system/reloader
Checking release: kube-system/sealed-secrets

Checking release: kube-system/storageclasses
    "VSAD": "GO0V", "Zone": "GZ"}, "OKE-tags": {"ClusterName": "oke.ETOKE-438.np.iad.go0v"}}'
  freeformTagsOverride: '{"ClusterName": "oke.ETOKE-438.np.iad.go0v"}'
↳ Found in kube-system/storageclasses

Checking release: kube-system/sysdig
      tags: 'cluster:oke.ETOKE-438.np.iad.go0v,vz-vsadid:GO0V,vz-vastid:'
    name: oke.ETOKE-438.np.iad.go0v
↳ Found in kube-system/sysdig


Checking release: kube-system/vertical-pod-autoscaler
Checking release: kube-system/vz-ingress-manager 



common data moudle

locals {
  # (TODO cappni7) Configuration values is a placeholder. Subnets will be confirmed with the network team
  # closer to the GA release date.
  environment_config = {
    "NONPROD" = {
      tenancy_name               = "vznprdbizgeneral"
      root_compartment_id        = "ocid1.tenancy.oc1..aaaaaaaaf3rzvq3we74v2a3zmgsbez734ziwpaquh62s3y3arp57tasaoxaq"
      platform_ingress_domain    = "ebiz.verizon.com"
      engineering_compartment_id = "ocid1.compartment.oc1..aaaaaaaag7milnrpu72yloqj3a4zbliskopvww64nb4kiqhp26qrlyjtiv4q"
      regions = {
        iad = {
          kms_vault_id = "ocid1.vault.oc1.iad.bbpa7dkdaaeuk.abuwcljtbexpr3thfwbn5opdupsqlsq4lknly744aygvhkrzd6sii65kh6bq"

          networking = {
            # oci.general.npd.us-ashburn-1.vcn
            vcn_id = "ocid1.vcn.oc1.iad.aaaaaaaajolhy6ptw26lngfs2aqaizynzsqj5nfr2cslamloumbze473fa5a"

            # oci.general.npd.us-ashburn-1.gz.oke.worker.subnet-2
            endpoint_subnet_id = "ocid1.subnet.oc1.iad.aaaaaaaahcpg2b2tvrxlm7asaofmitzfw54xkpxewgvcel4cr6hjbvlg66za"

            # oci.general.npd.us-ashburn-1.gz.oke.worker.subnet-2 
            node_subnet_id = "ocid1.subnet.oc1.iad.aaaaaaaahcpg2b2tvrxlm7asaofmitzfw54xkpxewgvcel4cr6hjbvlg66za"

            dns_nameservers = "153.114.241.7 159.67.63.7"

            # MASTER-OKE-ALL
            endpoint_nsgs = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaaid57bhj6o76ft2kfvzg6grjvnudfont56ldzdrjznn744mdy3lkq"]

            # MASTER-OKE-ALL
            node_nsgs = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaaid57bhj6o76ft2kfvzg6grjvnudfont56ldzdrjznn744mdy3lkq"]

            # oci.general.npd.us-ashburn-1.gz.general.subnet-1
            oci_lb_subnet_ids = ["ocid1.subnet.oc1.iad.aaaaaaaa5qhbqidcyflys6gmujxixnqa3ai2eq55sxrpmdbabtc5746sgnba"]

            # MASTER-HTTPS-ALL
            oci_lb_nsgs = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaalp3w4xhgmkbov2x663hf4hke3v4rs4ftvw3nhanawx4pwkcadqda"]
          }

        }
        phx = {
          kms_vault_id = "ocid1.vault.oc1.phx.a5ph4ro2aafqw.abyhqljtqxxwupzzfnudjtdaqyx4qgly3qdfdqhe7e7jtxyvqd2bvbqtxaea"
          networking = {
            # oci.general.npd.us-phoenix-1.vcn
            vcn_id = "ocid1.vcn.oc1.phx.aaaaaaaagkr5gcgjzhrsywhqguidnrytmzksut6tdzklc4cunavffeysrxpq"

            # oci.general.npd.us-phoenix-1.gz.oke.worker.subnet-1
            endpoint_subnet_id = "ocid1.subnet.oc1.phx.aaaaaaaabhleqcy2o7vg2qgcxmhardumw4gonvd6dlxfkvqoa6vuesy7wtiq"

            # oci.general.npd.us-phoenix-1.gz.oke.worker.subnet-1
            node_subnet_id = "ocid1.subnet.oc1.phx.aaaaaaaabhleqcy2o7vg2qgcxmhardumw4gonvd6dlxfkvqoa6vuesy7wtiq"

            dns_nameservers = "153.114.240.7 159.67.14.7"

            # MASTER-OKE-ALL
            endpoint_nsgs = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaaem36ojchg4xv4ydts7p6q37t7cfoemprmdim4bgp4qupxoza7ica"]

            # MASTER-OKE-ALL
            node_nsgs = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaaem36ojchg4xv4ydts7p6q37t7cfoemprmdim4bgp4qupxoza7ica"]

            # oci.general.npd.us-phoenix-1.gz.general.subnet-1
            oci_lb_subnet_ids = ["ocid1.subnet.oc1.phx.aaaaaaaanef5kzo5twnz6hxztfl4h7c6ncfzlnf3ypfl3xg3blkg5grichsa"]

            # MASTER-HTTPS-ALL
            oci_lb_nsgs = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaaswoc6tuqgdpfpohcpura26ts3jpetak7h6rtkco44pdzfxctb4sq"]
          }
        }
      }
      proxy = {
        http_proxy  = "http://proxy.ebiz.verizon.com:9290"
        https_proxy = "http://proxy.ebiz.verizon.com:9290"
        no_proxy    = "10.96.0.1,169.254.169.254,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com,svc.cluster.local."
      }
    }
    "PROD" = {
      tenancy_name               = "vzprodbizgeneral"
      root_compartment_id        = "ocid1.tenancy.oc1..aaaaaaaarmwlmudv2zgyb5edr4ifahsthejtpb2q2fcvifsugwqjsv4xzvwa"
      platform_ingress_domain    = "verizon.com"
      engineering_compartment_id = "ocid1.compartment.oc1..aaaaaaaabbkg52i3p6h42yyevanxbihp4myjtdzxskg2gtzlw3nzrohcwaxq"
      regions = {
        iad = {
          kms_vault_id = "ocid1.vault.oc1.iad.bbpe37cuaaeug.abuwcljsbj4t7rytlesxrnqygrvlc36zqony3ukaejwcf7eymxnfkferhuza"

          networking = {
            # oci.general.prd.us-ashburn-1.vcn
            vcn_id             = "ocid1.vcn.oc1.iad.aaaaaaaacu4pfrxovkzzjztb6v6fqmhhhscdqgffvp2qryq5araesjxmf77q"

            # oci.general.prd.us-ashburn-1.gz.oke.worker.subnet-1
            endpoint_subnet_id = "ocid1.subnet.oc1.iad.aaaaaaaawnjc2dnui6a7eg5qsejhrcqmlesfxgiycwddbtlbfr2n4szz35bq"

            # oci.general.prd.us-ashburn-1.gz.oke.worker.subnet-1
            node_subnet_id     = "ocid1.subnet.oc1.iad.aaaaaaaawnjc2dnui6a7eg5qsejhrcqmlesfxgiycwddbtlbfr2n4szz35bq"

            dns_nameservers    = "153.114.241.7 159.67.63.7"

            # MASTER-OKE-ALL
            endpoint_nsgs      = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaajzh224qlszmglqfptyz5klqqirbims2jkqoo4y5rzwjx77tkdjrq"]

            # MASTER-OKE-ALL
            node_nsgs          = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaajzh224qlszmglqfptyz5klqqirbims2jkqoo4y5rzwjx77tkdjrq"]

            # oci.general.prd.us-ashburn-1.gz.general.subnet-1
            oci_lb_subnet_ids  = ["ocid1.subnet.oc1.iad.aaaaaaaa6r3q46zolyvdwnnksrgqngunqv2nc2mfcehoaozqrzw6lgfghsca"]

            # MASTER-HTTPS-ALL
            oci_lb_nsgs        = ["ocid1.networksecuritygroup.oc1.iad.aaaaaaaaujg3p2q7llpbotnvqflrywtrwbwl4q3rxh2gjfpmxvsde3z6pr3a"]
          }

        }
        phx = {
          kms_vault_id = "ocid1.vault.oc1.phx.a5pe37fnaafqw.abyhqljt2micy6dlrahradjlos7lwygla2rexn6xqnz5eq7anzfkjlxoih2q"

          networking = {
            # oci.general.prd.us-phoenix-1.vcn
            vcn_id = "ocid1.vcn.oc1.phx.aaaaaaaasgpcc62ge67s3cywqpr2cknieiyapqpgir2ss6obkx7ib5soltvq"

            # oci.general.prd.us-phoenix-1.gz.oke.worker.subnet-1
            endpoint_subnet_id = "ocid1.subnet.oc1.phx.aaaaaaaaqohuelxafoevf2iuz3ric2234gdpvyfrgj3nxo3zbc32a5nyckkq"

            # oci.general.prd.us-phoenix-1.gz.oke.worker.subnet-1
            node_subnet_id = "ocid1.subnet.oc1.phx.aaaaaaaaqohuelxafoevf2iuz3ric2234gdpvyfrgj3nxo3zbc32a5nyckkq"

            dns_nameservers    = "153.114.240.7 159.67.14.7"

            # MASTER-OKE-ALL
            endpoint_nsgs      = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaarpftrze7pccepmnbgx6ofv2g54pwdujav43qjfszddemsa2cohna"]

            # MASTER-OKE-ALL
            node_nsgs          = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaarpftrze7pccepmnbgx6ofv2g54pwdujav43qjfszddemsa2cohna"]

            # oci.general.prd.us-phoenix-1.gz.general.subnet-1
            oci_lb_subnet_ids  = ["ocid1.subnet.oc1.phx.aaaaaaaay5jyp3t2e75jxb3vzuuitiiewff2f5enaglziepvft4wxzmkkacq"]

            # MASTER-HTTPS-ALL
            oci_lb_nsgs        = ["ocid1.networksecuritygroup.oc1.phx.aaaaaaaapvqwaduei4fq66fbaqi5mb7ilov33tfawtkh4ysdj6f5h3tunpsa"]
          }
        }
      }
      proxy = {
        http_proxy  = "http://vzproxy.verizon.com:9290"
        https_proxy = "http://vzproxy.verizon.com:9290"
        no_proxy    = "10.96.0.1,169.254.169.254,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com,svc.cluster.local."
      }
    }
  }
  cluster_config = {
    kubernetes_version = "v1.31.1"
    addons_overlay     = "release_1_31"
  }
  # Most Compartment names are the VSAD. This is a map of the exceptions.
  vsad_compartment_map = {
    "CKKV" = "Governance"
    "D0SV" = "Security"
    "GVGV" = "Engineering"
    "GO0V" = "K8"
  }
  compartment_name = coalesce(
    lookup(local.vsad_compartment_map, var.vsad, null),
    var.vsad
  )
  compartment_id = data.oci_identity_compartments.cluster.compartments[0].id

  environment_key = var.environment == "NONPROD" ? "np" : "pr"
  network_config  = local.environment_config[var.environment].regions[var.region_key].networking
  no_proxy = join(",", [local.environment_config[var.environment].proxy.no_proxy,
  data.oci_core_subnet.api_endpoint.cidr_block])
  region_id = data.oci_identity_regions.primary.regions[0].name
  zone      = upper(data.oci_core_subnet.node.defined_tags["Network-Tags.Zone"])

  vast_response   = var.enable_vast_service_request ? jsondecode(data.http.vast_services[0].response_body) : null
  vast_id         = local.vast_response != null ? (contains(keys(local.vast_response), "ApplicationID") ? local.vast_response.ApplicationID : "") : ""
  custodian_email = local.vast_response != null ? (contains(keys(local.vast_response), "CustodianEmail") ? local.vast_response.CustodianEmail : "") : ""
  portfolio       = local.vast_response != null ? (contains(keys(local.vast_response), "BusinessUnit") ? local.vast_response.BusinessUnit : "") : ""

  cluster_name = (
    var.cluster_id != null ?
    data.oci_containerengine_clusters.primary[0].clusters[0].name :
    join(".", [
      "oke",
      var.short_name,
      local.environment_key,
      var.region_key,
      lower(var.vsad)
    ])
  )

  # Use the UserID from the variable if provided; 
  # otherwise, retrieve it from the cluster's freeform tag
  user_id = (
    var.user_id == null ?
    data.oci_containerengine_clusters.primary[0].clusters[0].freeform_tags["UserID"] :
    var.user_id
  )
  whois_url = "https://orchestra.verizon.com/aws/api/whois"

  # Get the user metadata from the whois service
  # If the service is not enabled, use the email from the cluster's freeform tag
  user_metadata = (
    var.enable_whois_service_request ?
    jsondecode(data.http.user_metadata[0].response_body) :
    { mail = data.oci_containerengine_clusters.primary[0].clusters[0].freeform_tags["Owner"] }
  )

  # Filter keys by display name and defined tags
  # Expected display name format: VSAD-REGION-ENVIRONMENT
  # i.e. GO0V-ASH-NonProd
  kms_region = var.region_key == "iad" ? "ASH" : "PHX"
  kms_key_id = [
    for key in data.oci_kms_keys.primary.keys :
    key.id if lower(key.defined_tags["Compartment-tags.VSAD"]) == lower(var.vsad)
    && lower(regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).vsad) == lower(var.vsad)
    && regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).region == local.kms_region
    && lower(regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).env) == lower(var.environment)
  ][0]

  is_oke_coredns_addon_enabled = (
    var.cluster_id != null ?
    contains(data.oci_containerengine_addons.cluster[0].addons, "CoreDNS") :
    null
  )

  ## OCI Resource Tags
  tags = {

    block_storage = {
      defined = {
        "Block-Storage-tags.Expiration"   = ""
        "Block-Storage-tags.InstanceName" = local.cluster_name
        "Block-Storage-tags.UserID"       = lower(local.user_id)
        "Block-Storage-tags.VolumeGroup"  = ""
        "Block-Storage-tags.VSAD"         = upper(var.vsad)
        "Block-Storage-tags.Zone"         = local.zone
        "OKE-tags.ClusterName"            = local.cluster_name
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }

    cluster = {
      defined = null
      freeform = {
        "Owner"                = local.user_metadata.mail
        "UserID"               = lower(local.user_id)
        "VSAD"                 = upper(var.vsad)
        "VAST"                 = local.vast_id
        "UpdatedAt"            = formatdate("EEEE, DD-MMM-YYYY hh:mm:ss ZZZ", timestamp())
        "UpdatedPipelineUrl"   = data.external.pipeline_metadata.result.pipeline_url
        "UpdatedModuleVersion" = data.external.pipeline_metadata.result.oke_module_version
      }
    }

    fss = {
      defined = {
        "FSS-tags.Environment" = var.environment
        "FSS-tags.UserID"      = lower(local.user_id)
        "FSS-tags.VSAD"        = upper(var.vsad)
        "FSS-tags.Zone"        = local.zone
        "OKE-tags.ClusterName" = local.cluster_name
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }

    load_balancer = {
      defined = {
        "Load-Balancer-tags.Environment" = var.environment
        "Load-Balancer-tags.Owner"       = local.user_metadata.mail
        "Load-Balancer-tags.VSAD"        = upper(var.vsad)
        "Load-Balancer-tags.UserID"      = lower(local.user_id)
        "Load-Balancer-tags.Zone"        = local.zone
        "OKE-tags.ClusterName"           = local.cluster_name
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }
    node_pool = {
      defined = null
      freeform = {
        "ClusterName" = local.cluster_name
        "Owner"       = local.user_metadata.mail
        "UserID"      = lower(local.user_id)
        "VSAD"        = upper(var.vsad)
      }
    }

    node = {
      defined = {
        "Compute-Tag.AppCustodian" = local.custodian_email
        # (TODO cappni7) What should BornOn represent?
        # CAS will create nodes dynamically
        "Compute-Tag.BornOn"       = ""
        "Compute-Tag.Environment"  = var.environment
        "Compute-Tag.InstanceName" = local.cluster_name
        "Compute-Tag.InstanceRole" = "App"
        "Compute-Tag.LaunchedBy"   = local.user_metadata.mail
        "Compute-Tag.Live"         = "Build"
        "Compute-Tag.ManagedBy"    = local.user_metadata.mail
        "Compute-Tag.Portfolio"    = local.portfolio
        "Compute-Tag.RequestedBy"  = local.user_metadata.mail
        "Compute-Tag.TenancyName"  = local.environment_config[var.environment].tenancy_name
        "Compute-Tag.UserID"       = lower(local.user_id)
        "Compute-Tag.VAST"         = local.vast_id
        "Compute-Tag.VSAD"         = upper(var.vsad)
        "Compute-Tag.Zone"         = local.zone
        "OKE-tags.ClusterName"     = local.cluster_name
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }

    logging = {
      defined = null
      freeform = {
        "Owner"  = local.user_metadata.mail
        "UserID" = lower(local.user_id)
        "VSAD"   = upper(var.vsad)
        "VAST"   = local.vast_id
      }
    }
  }
}

data "http" "user_metadata" {
  count              = var.enable_whois_service_request ? 1 : 0
  url                = "${local.whois_url}/${lower(local.user_id)}?format=json"
  method             = "GET"
  request_timeout_ms = 7000
  retry {
    attempts     = 5
    max_delay_ms = 10000
    min_delay_ms = 2000
  }
}

data "http" "vast_services" {
  count              = var.enable_vast_service_request ? 1 : 0
  method             = "GET"
  request_timeout_ms = 8000
  url = join("", [
    "https://vast-services.verizon.com/itapm/getapplicationdetails?outputtype=json&vsadid=",
    lower(var.vsad)
  ])
  retry {
    attempts     = 9
    max_delay_ms = 60000
    min_delay_ms = 15000
  }
}

data "oci_containerengine_addons" "cluster" {
  count      = var.cluster_id != null ? 1 : 0
  cluster_id = var.cluster_id
}

data "oci_containerengine_clusters" "primary" {
  count          = var.cluster_id != null ? 1 : 0
  compartment_id = local.compartment_id
  filter {
    name   = "id"
    values = [var.cluster_id]
  }
}

data "oci_core_subnet" "node" {
  subnet_id = local.network_config.node_subnet_id
}

data "oci_core_subnet" "api_endpoint" {
  subnet_id = local.network_config.endpoint_subnet_id
}

data "oci_identity_compartments" "cluster" {
  compartment_id            = local.environment_config[var.environment].root_compartment_id
  compartment_id_in_subtree = true
  name                      = local.compartment_name
  lifecycle {
    postcondition {
      condition     = length(self.compartments) > 0
      error_message = "Compartment not found for VSAD '${var.vsad}'."
    }
  }
}

data "oci_identity_regions" "primary" {
  filter {
    name   = "key"
    values = [upper(var.region_key)]
  }
}

data "oci_kms_vault" "primary" {
  vault_id = local.environment_config[var.environment].regions[var.region_key]["kms_vault_id"]
}

data "oci_kms_keys" "primary" {
  compartment_id      = local.compartment_id
  management_endpoint = data.oci_kms_vault.primary.management_endpoint
}

data "external" "pipeline_metadata" {
  program = ["python", "${path.module}/scripts/pipeline_metadata.py"]
  query = {
    path_root = path.root
  }
}


