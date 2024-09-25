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
    
    
    
    
 
   maintenance_policy {
    recurring_window {
      start_time = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_start_time",
        var.maintenance_start_time
      )

      end_time = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_end_time",
        var.maintenance_end_time
      )

      recurrence = lookup(
        local.maintenance_window_overrides[local.cluster_key],
        "maintenance_recurrence",
        var.maintenance_recurrence
      )
    }
  }
  
  
    maintenance_window_overrides = {
    "<project_id>-<cluster_name>" = {
      "maintenance_start_time" = "xxx",
      "maintenance_end_time"    = "xxx",
      "maintenance_recurrence"  = "xxx"
    }
  }

  cluster_key = "<project_id>-<cluster_name>"