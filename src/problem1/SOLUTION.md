Provide your CLI command here:
```sh
 grep TSLA transaction-log.txt | jq -r ".order_id" | xargs -I {} curl -L https://example.com/api/{} >> ./output.txt
```

Explaination:
- `grep` to filter transaction contain keyword `TSLA`
- `jq` to parse each JSON payload and extract value from `order_id` key
- `xargs` map each extracted `order_id` to a `curl` command
- `curl` command send request to `https://example.com/api/:order_id` and write output to `./transaction-log.txt`
