for i in {1..100}; do
  curl -XPOST http://<alertmanager-ip>:9093/api/v1/alerts -H "Content-Type: application/json" -d '[
    {
      "labels": {
        "alertname": "TestAlert",
        "severity": "critical"
      },
      "annotations": {
        "summary": "Test alert"
      },
      "startsAt": "'$(date -Iseconds)'"
    }
  ]' &
done


 for i in {1..100}; do curl -s -XPOST http://<alertmanager-ip>:9093/api/v1/alerts -H "Content-Type: application/json" -d '[{"labels":{"alertname":"TestAlert","severity":"critical"},"annotations":{"summary":"Test alert"},"startsAt":"'"$(date -Iseconds)"'"}]' & done

ssh -L 9093:localhost:9093 <bastion-user>@<bastion-ip>


kubectl port-forward svc/prometheus-alertmanager 9093:9093 -n monitoring
