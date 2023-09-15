### Useful aliases

### kubectl

--sort-by
``` k get events -A --sort-by='{.metadata.creationTimestamp}'```

-l

--show-labels

--all-containers
```log --all=-containers```

kubectl config

raw kubectl 
`kubectl get --raw "/api/v1/nodes/<nodename>/proxy/configz" `
### Grep

-A1

-i

### Jsonpath

-ojsonpath='{.items[*].metadata.name}'