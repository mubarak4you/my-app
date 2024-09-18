/******************************************
  Create Cluster
 *****************************************/
resource "google_pubsub_topic" "topic" {
  provider     = google
  project      = var.project_id
  name         = local.cluster_name
  kms_key_name = local.kms_key
  labels       = local.cluster_labels
  lifecycle {
    precondition {
      condition = can(regex(
        "pubsub.googleapis.com",
        data.shell_script.google_services.output["enabled_services"]
      ))
      error_message = join(" ",
        ["The pubsub.googleapis.com service must be enabled for this project.",
        "Contact the Cloud Governance team for support."]
      )
    }
  }
}

resource "google_bigquery_dataset" "dataset" {
  provider                   = google
  project                    = var.project_id
  dataset_id                 = replace(local.cluster_name, "-", "_")
  location                   = "US"
  labels                     = local.cluster_labels
  delete_contents_on_destroy = true
  default_encryption_configuration {
    kms_key_name = local.kms_key_us
  }
  lifecycle {
    precondition {
      condition = can(regex(
        "bigquery.googleapis.com",
        data.shell_script.google_services.output["enabled_services"]
      ))
      error_message = join(" ",
        ["The bigquery.googleapis.com service must be enabled for this project.",
        "Contact the Cloud Governance team for support."]
      )
    }
  }
}

resource "google_container_cluster" "primary" {
  provider = google-beta

  name                = local.cluster_name
  location            = var.region
  node_locations      = local.node_locations
  deletion_protection = var.deletion_protection
  addons_config {
    horizontal_pod_autoscaling {
      disabled = false
    }
    http_load_balancing {
      disabled = false
    }
    network_policy_config {
      disabled = true
    }
    gcp_filestore_csi_driver_config {
      enabled = true
    }
    gce_persistent_disk_csi_driver_config {
      enabled = true
    }
    gke_backup_agent_config {
      enabled = true
    }
  }

  cluster_autoscaling {
    enabled = true
    resource_limits {
      resource_type = "cpu"
      minimum       = 6
      maximum       = 100000
    }
    resource_limits {
      resource_type = "memory"
      minimum       = 24
      maximum       = 800000
    }
    resource_limits {
      resource_type = "nvidia-tesla-k80"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-tesla-p100"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-tesla-p4"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-tesla-v100"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-tesla-t4"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-tesla-a100"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-a100-80gb"
      minimum       = 0
      maximum       = 10000
    }
    resource_limits {
      resource_type = "nvidia-l4"
      minimum       = 0
      maximum       = 10000
    }
    auto_provisioning_defaults {
      oauth_scopes = ["https://www.googleapis.com/auth/logging.write",
        "https://www.googleapis.com/auth/monitoring",
        "https://www.googleapis.com/auth/devstorage.read_only",
      "https://www.googleapis.com/auth/compute"]
      service_account   = data.shell_script.service_account.output["app_email"]
      boot_disk_kms_key = local.kms_key
      disk_size         = 100
      disk_type         = "pd-balanced"
      image_type        = "COS_CONTAINERD"
      shielded_instance_config {
        enable_integrity_monitoring = true
        enable_secure_boot          = true
      }
      management {
        auto_upgrade = true
        auto_repair  = true
      }
      upgrade_settings {
        max_surge       = 20
        max_unavailable = 0
      }
    }
    autoscaling_profile = "OPTIMIZE_UTILIZATION"
  }
  database_encryption {
    state    = "ENCRYPTED"
    key_name = local.kms_key
  }
  default_max_pods_per_node = 110
  enable_shielded_nodes     = true
  # Initial number of nodes per zone in the default pool
  initial_node_count = 3
  ip_allocation_policy {
    cluster_secondary_range_name  = local.cluster_secondary_range_name
    services_secondary_range_name = local.services_secondary_range_name
    pod_cidr_overprovision_config {
      disabled = false
    }
  }
  networking_mode = "VPC_NATIVE"
  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "APISERVER", "CONTROLLER_MANAGER", "SCHEDULER", "WORKLOADS"]
  }
  maintenance_policy {
    dynamic "recurring_window" {
      for_each = local.cluster_maintenance_window_is_recurring
      content {
        start_time = var.maintenance_start_time
        end_time   = var.maintenance_end_time
        recurrence = var.maintenance_recurrence
      }
    }

    dynamic "daily_maintenance_window" {
      for_each = local.cluster_maintenance_window_is_daily
      content {
        start_time = var.maintenance_start_time
      }
    }

    dynamic "maintenance_exclusion" {
      for_each = var.maintenance_exclusions
      content {
        exclusion_name = maintenance_exclusion.value.name
        start_time     = maintenance_exclusion.value.start_time
        end_time       = maintenance_exclusion.value.end_time

        dynamic "exclusion_options" {
          for_each = maintenance_exclusion.value.exclusion_scope == null ? [] : [maintenance_exclusion.value.exclusion_scope]
          content {
            scope = exclusion_options.value
          }
        }
      }
    }
  }
  master_authorized_networks_config {
    dynamic "cidr_blocks" {
      for_each = local.shared_subnet_cidr_blocks
      content {
        cidr_block   = cidr_blocks.value.cidr_block
        display_name = cidr_blocks.value.display_name
      }
    }
    gcp_public_cidrs_access_enabled = false
  }

  min_master_version = data.google_container_engine_versions.primary.release_channel_latest_version["STABLE"]
  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS", "APISERVER", "CONTROLLER_MANAGER", "SCHEDULER"]
    managed_prometheus {
      enabled = true
    }
    advanced_datapath_observability_config {
      enable_metrics = false
      enable_relay   = false
    }
  }
  network = local.network
  network_policy {
    enabled  = false
    provider = "PROVIDER_UNSPECIFIED"
  }
  node_config {
    disk_size_gb    = 100
    disk_type       = "pd-balanced"
    image_type      = "COS_CONTAINERD"
    labels          = local.cluster_labels
    resource_labels = local.cluster_resource_labels
    machine_type    = "e2-standard-4"
    metadata = {
      disable-legacy-endpoints = true
    }
    boot_disk_kms_key = local.kms_key
    service_account   = data.shell_script.service_account.output["app_email"]
    shielded_instance_config {
      enable_secure_boot          = true
      enable_integrity_monitoring = true
    }
  }

  notification_config {
    pubsub {
      enabled = true
      topic   = google_pubsub_topic.topic.id
    }
  }
  private_cluster_config {
    enable_private_endpoint     = true
    enable_private_nodes        = true
    private_endpoint_subnetwork = local.subnetwork
  }
  project = var.project_id
  release_channel {
    channel = "STABLE"
  }
  remove_default_node_pool = true
  resource_labels          = local.cluster_resource_labels
  cost_management_config {
    enabled = true
  }
  resource_usage_export_config {
    enable_network_egress_metering       = true
    enable_resource_consumption_metering = true
    bigquery_destination {
      dataset_id = google_bigquery_dataset.dataset.dataset_id
    }
  }
  subnetwork = local.subnetwork
  timeouts {
    create = lookup(var.timeouts, "create", "45m")
    update = lookup(var.timeouts, "update", "120m")
    delete = lookup(var.timeouts, "delete", "45m")
  }
  vertical_pod_autoscaling {
    enabled = true
  }
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }
  enable_intranode_visibility = true
  enable_l4_ilb_subsetting    = true
  datapath_provider           = "ADVANCED_DATAPATH"
  default_snat_status {
    disabled = false
  }
  lifecycle {
    ignore_changes = [
      cluster_telemetry,
      initial_node_count,
      master_auth,
      node_config,
      node_pool,
      resource_labels["mesh_id"]
    ]
    # ACM precondition
    precondition {
      condition = can(regex(
        "configmanagement",
        data.shell_script.fleet_features.output["enabled_features"]
      ))
      error_message = join(" ",
        ["The config management feature must be enabled on the fleet project.",
        "Contact the Cloud Governance team for support."]
      )
    }
    # ASM precondition
    precondition {
      condition = can(regex(
        "mesh.googleapis.com",
        data.shell_script.google_services.output["enabled_services"]
      ))
      error_message = join(" ",
        ["The mesh.googleapis.com service must be enabled for this project.",
        "Contact the Cloud Governance team for support."]
      )
    }
    precondition {
      condition = can(regex(
        "servicemesh",
        data.shell_script.fleet_features.output["enabled_features"]
      ))
      error_message = join(" ",
        ["The service mesh feature must be enabled on the fleet project.",
        "Contact the Cloud Governance team for support."]
      )
    }
    # (TODO cappni7) Need to verify if GKE will reject upgrades from less than master version N-1
    precondition {
      condition = can(regex(
        "container.googleapis.com",
        data.shell_script.google_services.output["enabled_services"]
      ))
      error_message = join(" ",
        ["The container.googleapis.com service must be enabled for this project.",
        "Contact the Cloud Governance team for support."]
      )
    }
    precondition {
      condition = can(regex(
        "^(NOT_FOUND|RUNNING)$",
        data.shell_script.cluster_status.output["status"]
      ))
      error_message = "The cluster must be in the RUNNING status to perform an update."
    }

    # precondition to check keyrings and keys
    precondition {
      condition = data.shell_script.kms_keys.output["kms_key_exists"] > 0
      error_message = join(" ",
        ["KMS key not found: ${local.kms_key}",
        "Contact the Cloud Governance team for support."]
      )
    }
    precondition {
      condition = data.shell_script.kms_keys.output["kms_key_us_exists"] > 0
      error_message = join(" ",
        ["KMS key not found: ${local.kms_key_us}",
        "Contact the Cloud Governance team for support."]
      )
    }
    precondition {
      condition = length(data.shell_script.service_account.output["app_email"]) > 0
      error_message = join(" ",
        ["Application service account not found.",
        "Contact the Cloud Governance team for support."]
      )
    }
    precondition {
      condition = length(data.shell_script.service_account.output["monitoring_email"]) > 0
      error_message = join(" ",
        ["Monitoring service account not found.",
        "Contact the Cloud Governance team for support."]
      )
    }
    postcondition {
      condition     = regex(local.semver_regex, self.master_version).minor <= local.module_minor_version
      error_message = "Cannot downgrade the cluster control plane minor version."
    }
    postcondition {
      condition = local.module_minor_version - regex(local.semver_regex, self.master_version).minor < 2
      error_message = join(" ",
        ["Must upgrade the cluster control plane by one minor version.",
        "For example 1.26 upgrades to 1.27."]
      )
    }
  }
  provisioner "local-exec" {
    # Run overprovisioning job to get node auto-provisioning (NAP) to create larger nodes
    # for new clusters. This greatly improves the start up time for new clusters
    # that exclusively use NAP. This provisioner will only run when a cluster
    # resource is created.
    environment = {
      "CLUSTER_NAME" = local.cluster_name
      "http_proxy"   = local.http_proxy
      "HTTP_PROXY"   = local.http_proxy
      "https_proxy"  = local.http_proxy
      "HTTPS_PROXY"  = local.http_proxy
      "no_proxy"     = local.no_proxy
      "NO_PROXY"     = local.no_proxy
      "PROJECT"      = var.project_id
      "REGION"       = var.region
    }
    working_dir = "${path.module}/scripts"
    interpreter = ["/bin/bash", "-c"]
    command     = "./run-overprovisioning.sh"
  }
}

# This will manually trigger node pool upgrades when the Kubernetes
# control plane version has changed. This is required because the
# google_container_cluster TF resource does not intitiate node pool kubernetes
# version upgrades
resource "terraform_data" "node_pools" {
  triggers_replace = google_container_cluster.primary.min_master_version
  provisioner "local-exec" {
    environment = {
      "CLUSTER_NAME"   = local.cluster_name
      "PROJECT"        = var.project_id
      "REGION"         = var.region
      "MASTER_VERSION" = data.google_container_engine_versions.primary.release_channel_latest_version["STABLE"]
    }
    working_dir = "${path.module}/scripts"
    interpreter = ["/bin/bash", "-c"]
    command     = "./upgrade-nodepools.sh"
  }
}
