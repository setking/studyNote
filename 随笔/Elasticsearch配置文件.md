✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  b52tHMyvA4XNKllq5lHK

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  5461dda33762315617afbdb8bcd79f5053a000f8b1295b3561a8ceefe9e9b596

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTcyLjE5LjAuMjo5MjAwIl0sImZnciI6IjU0NjFkZGEzMzc2MjMxNTYxN2FmYmRiOGJjZDc5ZjUwNTNhMDAwZjhiMTI5NWIzNTYxYThjZWVmZTllOWI1OTYiLCJrZXkiOiJqa3RGaDVnQmxQWTNfUS1SMV9EejpkRlU0MnRUb011c3ZOZFZ4VV93UjF3In0=

ℹ️ Configure other nodes to join this cluster:
• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTcyLjE5LjAuMjo5MjAwIl0sImZnciI6IjU0NjFkZGEzMzc2MjMxNTYxN2FmYmRiOGJjZDc5ZjUwNTNhMDAwZjhiMTI5NWIzNTYxYThjZWVmZTllOWI1OTYiLCJrZXkiOiJYV3psaEpnQm5kZXFheHNLbno2cDpwNVV6c19fVEp4V3YzUWpEVDlNbG53In0=

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.19.0`

###ElasticSearch dev-tool - 批量操作 bulk在chrome浏览器会报语法错误

