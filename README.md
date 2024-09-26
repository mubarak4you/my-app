
│ Error: Invalid index
│ 
│   on cluster.tf line 180, in resource "google_container_cluster" "primary":
│  180:         ])[0]["maintenance_start_time"],
│     ├────────────────
│     │ local.cluster_key is a string
│     │ local.maintenance_window_overrides is object with no attributes
│ 
│ The given key does not identify an element in this collection value: the
│ collection has no elements.


  cluster_key                             = "${var.project_id}-${local.cluster_name}"