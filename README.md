  shared_subnet_cidr_blocks = concat(
    local.shared_subnet_cidr_blocks_east,
    local.shared_subnet_cidr_blocks_west,
    local.project_cidr_blocks
  )