# WarpStream Fly.io Template

This repository contains a template for deploying [WarpStream Agents](https://docs.warpstream.com/warpstream/) to [fly.io](https://fly.io/).

# Instructions

1. [Sign up for a WarpStream account](https://console.warpstream.com/).
2. [Sign up for a fly.io account and download flyctl](https://fly.io/).
3. Clone this repository and cd into it.
4. Review and edit `fly.toml` to fill in your specific details. At the very least you should:
    1. Pick a name for your WarpStream cluster and put it in the value of `app`.
    2. Edit `WARPSTREAM_DISCOVERY_KAFKA_HOSTNAME_OVERRIDE` to use the same value as `app`. For example if your `app` name is `warpstream_fly_123` then the value of `WARPSTREAM_DISCOVERY_KAFKA_HOSTNAME_OVERRIDE` should be `warpstream_fly_123.fly.dev`.
    3. Determine which fly.io region you want to deploy to and set that in `primary_region`.
    4. Fill in the value of `WARPSTREAM_DEFAULT_VIRTUAL_CLUSTER_ID` from the [WarpStream Admin Console](https://console.warpstream.com/virtual_clusters).
    5. Fill in the value of `WARPSTREAM_BUCKET_URL`. If you don't have an object store / bucket selected already, we've found that Cloudflare R2 works well with fly.io. We have more documentation on how to set that up [here](https://docs.warpstream.com/warpstream/reference/use-the-agents-with-an-s3-compatible-object-store-minio-oracle-cloud-etc).
5. Run `flyctl launch` to create your WarpStream cluster.
6. Run `flyctl deploy` to make sure your cluster is deployed.
7. Run `flyctl ips allocate-v4` to create a public IP4 address at which your WarpStream cluster can be reached.
8. Run `fly secrets set AWS_ACCESS_KEY_ID=XXXXXXXX` with whatever the value of your object store access key is.
9. Run `fly secrets set AWS_SECRET_ACCESS_KEY=XXXXXXXX` with whatever the value of your object store secret access key is.
10. Run `fly secrets set WARPSTREAM_API_KEY=XXXXXXXX` with your [WarpStream API key from the admin console](https://console.warpstream.com/api_keys).
11. Run `fly scale show` to see what machines your WarpStream Agents are running on.
12. The flyctl default machines are a bit too underpowered for the WarpStream Agents, so run `fly scale vm performance-2x` to get a bit more CPU and RAM.
13. If you want high availability, run `fly scale count 2` or `fly scale count 3` to run more than one instance of the WarpStream Agents.

Congrats! You have a fully functional WarpStream cluster running on fly.io now backed by Cloudflare R2 object storage.

Before you attempt to use the cluster, make sure that authentication is properly enabled. Run the following command:

```
warpstream kcmd -type diagnose-connection -bootstrap-host $YOUR_APP_NAME.fly.dev -tls
```

You should an error about how SASL authentication failed which means your cluster is properly enforcing authentication. If you see success, DO NOT USE THIS CLUSTER AS IT IS NOT PROPERLY ENFORCING AUTHENTICATION.

Now that we've confirmed authentication is enforced, navigate to the [WarpStream Admin Console](https://console.warpstream.com/virtual_clusters) and [create a new set of credentials for this cluster](https://docs.warpstream.com/warpstream/how-to/configure-authentication-for-the-warpstream-agent).

Once you have the credentials, try the `diagnose-connection` command again, but this time with credentials:

```
warpstream kcmd -type diagnose-connection -bootstrap-host warpstream-agent-demo.fly.dev -tls -username ccun_XXXXXXXX -password ccp_XXXXXXXX
```

This time you should get a successful response. That means your WarpStream cluster is ready to go!