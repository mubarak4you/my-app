curl -u "admin:<your-admin-password>" -o /dev/null -s -w "HTTP status: %{http_code}\n" "http://<grafana-url>/d/<dashboard-uid>/<dashboard-name>"





<your-admin-password> with your Grafana admin password.
<grafana-url> with your Grafana URL.
<dashboard-uid> with the unique ID of your dashboard.
<dashboard-name> with the name of your dashboard.