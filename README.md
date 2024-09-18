/******************************************
  Define locals and data resources
 *****************************************/

locals {
  environment                             = var.environment != "" ? var.environment : regex("^vz-(?P<group>it|nonit)-(?P<environment>np|pr)-(?P<vsad>[a-z0-9]+)-.*", var.project_id).environment
  vsad                                    = var.vsad != "" ? var.vsad : regex("^vz-(?P<group>it|nonit)-(?P<environment>np|pr)-(?P<vsad>[a-z0-9]+)-.*", var.project_id).vsad
  bu                                      = lookup(data.google_project.project.labels, "bu", "BU not found")
  cluster_name                            = lower("gke-${var.name}-${local.environment}-${var.region}-${local.vsad}")
  node_locations                          = var.region == "us-east4" ? ["us-east4-a", "us-east4-b"] : ["us-west1-a", "us-west1-b"]
  kms_project                             = "vz-it-${local.environment}-d0sv-vsadkms-0"
  key_prefix                              = join("-", regex("^(vz)-(it|nonit)-(np|pr)", var.project_id))
  keyring_name                            = "${local.key_prefix}-kr-${local.bu}"
  key_name                                = "${local.key_prefix}-kms-${local.vsad}"
  keyring                                 = "projects/${local.kms_project}/locations/${var.region}/keyRings/${local.keyring_name}"
  keyring_us                              = "projects/${local.kms_project}/locations/us/keyRings/${local.keyring_name}"
  kms_key                                 = "${local.keyring}/cryptoKeys/${local.key_name}"
  kms_key_us                              = "${local.keyring_us}/cryptoKeys/${local.key_name}"
  cluster_maintenance_window_is_recurring = var.maintenance_recurrence != "" && var.maintenance_end_time != "" ? [1] : []
  cluster_maintenance_window_is_daily     = length(local.cluster_maintenance_window_is_recurring) > 0 ? [] : [1]
  hub_project_id                          = var.hub_project_id == "" ? var.project_id : var.hub_project_id
  gke_hub_membership_name                 = trimsuffix(substr(local.cluster_name, 0, 63), "-")
  module_minor_version                    = regex("^(?P<major>[0-9]+).(?P<minor>[0-9]+)", var.kubernetes_version).minor
  semver_regex                            = "(?P<major>[0-9]+).(?P<minor>[0-9]+).(?P<patch>[0-9]+)"

  # VPC Network
  sharedvpc_project             = local.environment == "pr" ? "vz-it-pr-exhv-sharedvpc-228020" : "vz-it-np-exhv-sharedvpc-228116"
  sharedvpc_name                = var.region == "us-east4" ? "shared-${local.environment}-east" : "shared-${local.environment}-west"
  node_subnet_name              = var.region == "us-east4" ? "shared-${local.environment}-east-gke-nodes-1" : "shared-${local.environment}-west-gke-nodes-1"
  cluster_secondary_range_name  = var.region == "us-east4" ? "shared-${local.environment}-east-gke-pod-0" : "shared-${local.environment}-west-gke-pod-0"
  services_secondary_range_name = var.region == "us-east4" ? "shared-${local.environment}-east-gke-services-0" : "shared-${local.environment}-west-gke-services-0"
  network                       = "projects/${local.sharedvpc_project}/global/networks/${local.sharedvpc_name}"
  subnetwork                    = "projects/${local.sharedvpc_project}/regions/${var.region}/subnetworks/${local.node_subnet_name}"

  # HTTP proxy
  http_proxy        = lower(local.environment) == "pr" ? "http://vzproxy.verizon.com:9290" : "http://proxy.ebiz.verizon.com:9290"
  static_no_proxy   = "172.20.0.1,10.100.0.1,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com,10.74.132.65,192.168.0.1,.googleapis.com"
  node_subnet_range = data.google_compute_subnetwork.subnet.ip_cidr_range
  pod_subnet_range = data.google_compute_subnetwork.subnet.secondary_ip_range[
    index(
      data.google_compute_subnetwork.subnet.secondary_ip_range[*].range_name,
      local.cluster_secondary_range_name
  )].ip_cidr_range
  svc_subnet_range = data.google_compute_subnetwork.subnet.secondary_ip_range[
    index(
      data.google_compute_subnetwork.subnet.secondary_ip_range[*].range_name,
      local.services_secondary_range_name
  )].ip_cidr_range
  no_proxy = format("%s,%s,%s,%s", local.static_no_proxy, local.node_subnet_range, local.pod_subnet_range, local.svc_subnet_range)

  # Master authorized networks
  shared_subnet_cidr_blocks_east = [for subnet in data.google_compute_subnetworks.east.subnetworks :
    {
      cidr_block   = subnet.ip_cidr_range
      display_name = subnet.name
    }
    # cappni7 - Temporarily restrict to east-green-subnet-1 until privately used public IP (PUPI) is
    # supported by MAN
    if can(regex("^shared-${local.environment}-east-green-subnet-1", subnet.name))
  ]
  shared_subnet_cidr_blocks_west = [for subnet in data.google_compute_subnetworks.west.subnetworks :
    {
      cidr_block   = subnet.ip_cidr_range
      display_name = subnet.name
    }
    # cappni7 - Temporarily restrict to west-green-subnet-1 until privately used public IP (PUPI) is
    # supported by MAN
    if can(regex("^shared-${local.environment}-west-green-subnet-1", subnet.name))
  ]

  # Define additional authorized networks for user projects
  project_authorized_networks = {
    # "vz-it-np-go0v-dev-gketst-0" = [
    #   {
    #     cidr_block   = "192.168.0.0/16"
    #     display_name = "Test authorized network"
    #   }
    # ]
  }
  project_cidr_blocks = flatten([
    for k, v in local.project_authorized_networks :
    v if k == var.project_id
  ])
  shared_subnet_cidr_blocks = concat(
    local.shared_subnet_cidr_blocks_east,
    local.shared_subnet_cidr_blocks_west,
    local.project_cidr_blocks
  )

  # Labels
  env_long_name = local.environment == "pr" ? "production" : "nonproduction"
  owner         = lower(replace(replace(var.user_email, "@", "__"), ".", "-"))
  ccdistro      = "gke-platform-engineering_verizon-com"
  cluster_labels = {
    bu                       = lower(local.bu)
    ccdistro                 = local.ccdistro
    env                      = local.env_long_name
    owner                    = local.owner
    userid                   = lower(var.user_id)
    vsad                     = local.vsad
    infra_rootsync_branch    = lower(replace(replace(var.config_sync_infra_branch, "/", "_"), ".", "-"))
    security_rootsync_branch = lower(replace(replace(var.config_sync_security_branch, "/", "_"), ".", "-"))
    last_pipeline_id         = var.pipeline_id
    cluster_module_version   = lower(replace(var.cluster_module_version, ".", "-"))
  }
  cluster_resource_labels = merge(
    local.cluster_labels,
    { "mesh_id" = "proj-${data.google_project.project.number}" }
  )

  # Cluster output
  cluster_id                      = google_container_cluster.primary.id
  cluster_endpoint                = google_container_cluster.primary.endpoint
  cluster_region                  = var.region
  cluster_master_version          = google_container_cluster.primary.master_version
  cluster_min_master_version      = google_container_cluster.primary.min_master_version
  cluster_release_channel         = google_container_cluster.primary.release_channel
  cluster_master_auth             = concat(google_container_cluster.primary[*].master_auth, [])
  cluster_master_auth_list_layer1 = local.cluster_master_auth
  cluster_master_auth_list_layer2 = local.cluster_master_auth_list_layer1[0]
  cluster_master_auth_map         = local.cluster_master_auth_list_layer2[0]
  cluster_ca_certificate          = local.cluster_master_auth_map["cluster_ca_certificate"]
  hub_membership_id               = google_gke_hub_membership.primary.membership_id
}

data "google_project" "project" {
  provider   = google
  project_id = var.project_id
}

data "google_compute_subnetwork" "subnet" {
  provider  = google
  self_link = "https://www.googleapis.com/compute/v1/projects/${local.sharedvpc_project}/regions/${var.region}/subnetworks/${local.node_subnet_name}"
}

data "google_compute_subnetworks" "east" {
  provider = google
  project  = local.sharedvpc_project
  region   = "us-east4"
}

data "google_compute_subnetworks" "west" {
  provider = google
  project  = local.sharedvpc_project
  region   = "us-west1"
}

data "google_container_engine_versions" "primary" {
  provider = google
  location = var.region
  project  = var.project_id
  # Add suffix to ensure the correct MINOR or PATCH version is resoled. For example, "1.1" should not resolve "1.15"
  version_prefix = can(regex("\\d\\.\\d+\\.\\d", var.kubernetes_version)) ? "${var.kubernetes_version}-" : "${var.kubernetes_version}."
}

# We are using the shell_script provider to create data sources
# to get information that is not available using the regular
# Google TF provider data sources.
data "shell_script" "cluster_status" {
  lifecycle_commands {
    read = file("${path.module}/scripts/get-cluster-status.sh")
  }
  environment = {
    "PROJECT"      = var.project_id
    "CLUSTER_NAME" = local.cluster_name
    "REGION"       = var.region
  }
  interpreter       = ["/bin/bash", "-c"]
  working_directory = "${path.module}/scripts"
}
data "shell_script" "google_services" {
  lifecycle_commands {
    read = file("${path.module}/scripts/list-services.sh")
  }
  environment = {
    "GOOGLE_PROJECT" = var.project_id
  }

  interpreter       = ["/bin/bash", "-c"]
  working_directory = "${path.module}/scripts"
}

data "shell_script" "service_account" {
  lifecycle_commands {
    read = file("${path.module}/scripts/get-service-account.sh")
  }
  environment = {
    "PROJECT" = var.project_id
    "VSAD"    = local.vsad
  }

  interpreter       = ["/bin/bash", "-c"]
  working_directory = "${path.module}/scripts"
}

data "shell_script" "kms_keys" {
  lifecycle_commands {
    read = file("${path.module}/scripts/check-kms-keys.sh")
  }
  environment = {
    "KEYRING"    = local.keyring
    "KEY"        = local.kms_key
    "KEYRING_US" = local.keyring_us
    "KEY_US"     = local.kms_key_us
  }

  interpreter       = ["/bin/bash", "-c"]
  working_directory = "${path.module}/scripts"
}

data "shell_script" "fleet_features" {
  lifecycle_commands {
    read = file("${path.module}/scripts/list-fleet-features.sh")
  }
  environment = {
    "PROJECT" = var.project_id
  }

  interpreter       = ["/bin/bash", "-c"]
  working_directory = "${path.module}/scripts"
}
