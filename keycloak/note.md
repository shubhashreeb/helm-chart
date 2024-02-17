
curl -X POST \
  'http://10.99.24.16:80/realms/nshub/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&client_id=auth-svc&client_secret=3nUB5EUG3fenYFa9xqnF376PLFZHWxFV&username=aaron&password=aaron' -v

