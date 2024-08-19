locals {
  # (TODO cappni7) Configuration values is a placeholder. Subnets will be confirmed with the network team
  # closer to the GA release date.
  environment_config = {
    "NONPROD" = {
      tenancy_name        = "vznprdbizgeneral"
      root_compartment_id = "ocid1.tenancy.oc1..aaaaaaaaf3rzvq3we74v2a3zmgsbez734ziwpaquh62s3y3arp57tasaoxaq"
      network_config = {
        iad = {
          vcn_id               = "ocid1.vcn.oc1.iad.aaaaaaaajolhy6ptw26lngfs2aqaizynzsqj5nfr2cslamloumbze473fa5a"    # oci.general.npd.us-ashburn-1.vcn
          endpoint_subnet_id   = "ocid1.subnet.oc1.iad.aaaaaaaahcpg2b2tvrxlm7asaofmitzfw54xkpxewgvcel4cr6hjbvlg66za" # oci.general.npd.us-ashburn-1.gz.oke.worker.subnet-2
          service_lb_subnet_id = "ocid1.subnet.oc1.iad.aaaaaaaa5qhbqidcyflys6gmujxixnqa3ai2eq55sxrpmdbabtc5746sgnba" # oci.general.npd.us-ashburn-1.gz.general.subnet-1
          node_subnet_id       = "ocid1.subnet.oc1.iad.aaaaaaaahcpg2b2tvrxlm7asaofmitzfw54xkpxewgvcel4cr6hjbvlg66za" # oci.general.npd.us-ashburn-1.gz.oke.worker.subnet-2
          dns_nameservers      = "153.114.241.7 159.67.63.7"
        }
        phx = {
          vcn_id               = ""
          endpoint_subnet_id   = ""
          service_lb_subnet_id = ""
          node_subnet_id       = ""
          dns_nameservers      = "153.114.240.7 159.67.14.7"
        }
      }
      proxy = {
        http_proxy  = "http://proxy.ebiz.verizon.com:9290"
        https_proxy = "http://proxy.ebiz.verizon.com:9290"
        no_proxy    = "10.96.0.1,169.254.169.254,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com"
      }
      kms = {
        iad = {
          vault_id = "ocid1.vault.oc1.iad.bbpa7dkdaaeuk.abuwcljtbexpr3thfwbn5opdupsqlsq4lknly744aygvhkrzd6sii65kh6bq"
        }
        phx = {
          vault_id = "ocid1.vault.oc1.phx.a5ph4ro2aafqw.abyhqljtqxxwupzzfnudjtdaqyx4qgly3qdfdqhe7e7jtxyvqd2bvbqtxaea"
        }
      }
    }
    "PROD" = {
      tenancy_name        = ""
      root_compartment_id = ""
      network_config = {
        iad = {
          vcn_id               = ""
          endpoint_subnet_id   = ""
          service_lb_subnet_id = ""
          node_subnet_id       = ""
          dns_nameservers      = "153.114.241.7 159.67.63.7"
        }
        phx = {
          vcn_id               = ""
          endpoint_subnet_id   = ""
          service_lb_subnet_id = ""
          node_subnet_id       = ""
          dns_nameservers      = "153.114.240.7 159.67.14.7"
        }
      }
      proxy = {
        http_proxy  = "http://vzproxy.verizon.com:9290"
        https_proxy = "http://vzproxy.verizon.com:9290"
        no_proxy    = "10.96.0.1,169.254.169.254,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com"
      }
      kms = {
        iad = {
          vault_id = ""
        }
        phx = {
          vault_id = ""
        }
      }
    }
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
  network_config  = local.environment_config[var.environment].network_config[var.region_key]
  no_proxy = join(",", [local.environment_config[var.environment].proxy.no_proxy,
  data.oci_core_subnet.api_endpoint.cidr_block])
  region_id = data.oci_identity_regions.primary.regions[0].name
  zone      = upper(data.oci_core_subnet.node.defined_tags["Network-Tags.Zone"])

  # Get the VAST ID from a tag if we can because the VAST service is unreliable.
  vast_id = (
    var.cluster_id == null ?
    jsondecode(data.http.vast_services[0].response_body).ApplicationID :
    data.oci_containerengine_clusters.primary[0].clusters[0].freeform_tags["VAST"]
  )
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

  # Get user metadata from Orchestra
  user_id = (
    var.cluster_id != null ?
    data.oci_containerengine_clusters.primary[0].clusters[0].freeform_tags["UserID"] :
    var.user_id
  )
  whois_url        = "https://orchestra.verizon.com/aws/api/whois"
  user_metadata    = jsondecode(data.http.user_metadata.response_body)
  manager_metadata = jsondecode(data.http.manager_metadata.response_body)

  # Filter keys by display name and defined tags
  # Expected display name format: VSAD-REGION-ENVIRONMENT
  # i.e. GO0V-ASH-NonProd
  kms_region = var.region_key == "iad" ? "ASH" : "PHX"
  kms_key_id = [
    for key in data.oci_kms_keys.primary.keys :
    key.id if lower(key.defined_tags["KMS-tags.VSAD"]) == lower(var.vsad)
    && lower(key.defined_tags["KMS-tags.Environment"]) == lower(var.environment)
    && lower(regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).vsad) == lower(var.vsad)
    && regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).region == local.kms_region
    && lower(regex("^(?P<vsad>[A-Z0-9]+)-(?P<region>ASH|PHX)-(?P<env>[A-Za-z]+)$", key.display_name).env) == lower(var.environment)
  ][0]

  is_oke_coredns_addon_enabled = (
    var.cluster_id != null ?
    contains(data.oci_containerengine_addons.cluster[0].addons, "CoreDNS") :
    null
  )
  node_image_type = (
    var.cluster_id != null ?
    data.oci_containerengine_clusters.primary[0].clusters[0].freeform_tags["ComputeOS"] :
    null
  )
  is_oke_image_type = local.node_image_type == "OKE"
  dns_forward = (
    local.is_oke_image_type ?
    local.environment_config[var.environment].network_config[var.region_key].dns_nameservers :
    "/etc/resolv.conf"
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
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }

    cluster = {
      defined = null
      freeform = {
        "Owner"  = local.user_metadata.mail
        "UserID" = lower(local.user_id)
        "VSAD"   = upper(var.vsad)
        "VAST"   = local.vast_id
      }
    }

    fss = {
      defined = {
        "FSS-tags.Environment" = var.environment
        "FSS-tags.UserID"      = lower(local.user_id)
        "FSS-tags.VSAD"        = upper(var.vsad)
        "FSS-tags.Zone"        = local.zone
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
        "Compute-Tag.AppCustodian" = local.manager_metadata.mail
        # (TODO cappni7) What should BornOn represent?
        # CAS will create nodes dynamically
        "Compute-Tag.BornOn"       = ""
        "Compute-Tag.Environment"  = var.environment
        "Compute-Tag.InstanceName" = local.cluster_name
        "Compute-Tag.InstanceRole" = "App"
        "Compute-Tag.LaunchedBy"   = local.user_metadata.mail
        "Compute-Tag.Live"         = "Build"
        "Compute-Tag.ManagedBy"    = local.user_metadata.mail
        "Compute-Tag.Portfolio"    = ""
        "Compute-Tag.RequestedBy"  = local.user_metadata.mail
        "Compute-Tag.TenancyName"  = local.environment_config[var.environment].tenancy_name
        "Compute-Tag.UserID"       = lower(local.user_id)
        "Compute-Tag.VAST"         = local.vast_id
        "Compute-Tag.VSAD"         = upper(var.vsad)
        "Compute-Tag.Zone"         = local.zone
      }
      freeform = {
        "ClusterName" = local.cluster_name
      }
    }
  }
}

data "http" "user_metadata" {
  url                = "${local.whois_url}/${lower(local.user_id)}?format=json"
  method             = "GET"
  request_timeout_ms = 5000
  retry {
    attempts     = 2
    max_delay_ms = 10000
    min_delay_ms = 2000
  }
}

data "http" "manager_metadata" {
  url                = "${local.whois_url}/${lower(local.user_metadata.managerId)}?format=json"
  method             = "GET"
  request_timeout_ms = 5000
  retry {
    attempts     = 2
    max_delay_ms = 10000
    min_delay_ms = 2000
  }
}
data "http" "vast_services" {
  count              = var.cluster_id == null ? 1 : 0
  method             = "GET"
  request_timeout_ms = 8000
  url = join("", [
    "https://vast-services.verizon.com/apm/getapplicationdetails?outputtype=json&vsadid=",
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
  vault_id = local.environment_config[var.environment]["kms"][var.region_key]["vault_id"]
}

data "oci_kms_keys" "primary" {
  compartment_id      = local.compartment_id
  management_endpoint = data.oci_kms_vault.primary.management_endpoint
}
