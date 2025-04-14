helm get values prometheus -n kube-system -o yaml | grep -A6 -E '^(alertmanager|server|prometheus-node-exporter):' | grep -E '^(alertmanager|server|prometheus-node-exporter):|resources:|cpu:|memory:'
