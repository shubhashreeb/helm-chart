
Test the deployment -> 

curl http://192.168.86.210:30080/v1/GetRateLimits \
  --header 'Content-Type: application/json' \
  --data '{
    "requests": [
        {
            "name": "requests_per_sec",
            "uniqueKey": "account:12345",
            "hits": "10",
            "limit": "10",
            "duration": "1000"
        }
    ]
}'