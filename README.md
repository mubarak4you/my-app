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



