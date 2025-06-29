# Unleash Edge Quickstart Guide

**Get up and running with Unleash Edge in under 5 minutes**

## Why Use Unleash Edge

Unleash Edge is a high-performance proxy that sits between your Unleash server and your SDKs. Your users get faster responses, your infrastructure stays resilient, and your team gets one less thing to worry about at 3 AM. It's like having a backup generator for your feature flags—everything keeps running even when the main power goes out!

Here's why you might need it:

**Performance**: A single Unleash server handles tremendous load, but what if you need even more? By deploying an Edge node before the main server, you're using in-memory caching and Rust's zero-cost abstractions to support thousands of connected SDKs without increasing load on your primary Unleash instance. It's perfect for high-traffic applications!

**Global Scale**: Keep the "last mile" short! Users in Texas shouldn't wait for responses from your `europe-west-1` Unleash server. We can't beat the speed of light (yet), but deploying Edge instances in every region you operate definitely helps the application run faster!

**Resilience**: Edge continues serving feature flags even when your Unleash server is unreachable or goes down. Your applications stay functional during outages, deployments, or network hiccups between regions.

<img width="706" alt="image" src="https://github.com/user-attachments/assets/2333cb5f-84d5-4244-b305-0b8dd890bb9f" />

## Quickstart

With this quickstart, you get it running in a minute. We promise it's simple—if you have a PhD in distributed systems, you won't need it! Notice, it's not production-ready, but it's perfect for getting your hands dirty and seeing Edge in action.

### Prerequisites

For this quickstart, we assume you already have the following:

- Docker and Docker Compose installed
- Running Unleash server (using your own or the provided [docker-compose setup](https://github.com/Unleash/unleash/blob/main/docker-compose.yml))
- An application with Unleash SDK already configured (we use [React SDK example](https://github.com/Unleash/unleash-sdk-examples/tree/main/React), but you can go with your own)
- A feature flag or two that your application can demonstrate

<img width="848" alt="image" src="https://github.com/user-attachments/assets/ab5836bb-ced9-45d9-9f85-e522da2a4929" />

## Setup Steps

### Step 1: Create an API Token

Before launching Edge, create an API token through the Unleash UI:

1. Open your Unleash instance at `http://localhost:4242`
2. Navigate to `Admin → Access Control → API Access`
3. Click "New API Token"
4. Select "Server-side SDK (Client)" as token type
5. Choose the desired project (We proceed with "ALL")
6. Choose the desired environment (We stick to the "development")
7. Create the token
8. Copy the generated token (you'll need it in the next step)

<img width="714" alt="image" src="https://github.com/user-attachments/assets/58110919-86f3-46b6-b53a-66cdf97fc0a0" />

### Step 2: Add Edge to Your Docker Compose

Update your existing `docker-compose.yml` file by adding the Edge service:

```yaml
services:
  # ... your existing server and db ...

  edge:
    image: unleashorg/unleash-edge:latest
    ports:
      - "3063:3063"
    environment:
      # Enable debug logging to see what's happening
      RUST_LOG: "warn,unleash_edge=debug"
      # Point to the Unleash server (same network, so use service name)
      UPSTREAM_URL: "http://web:4242"
      # Replace with your actual client token from Step 1
      TOKENS: "your-actual-token-here"
      # Enable strict mode
      STRICT: "true"
    command: edge
```

### Step 3: Launch the Updated Stack

```bash
# Start everything including Edge
docker-compose up -d edge
```

## Verification

### Check Edge Logs

```bash
# Watch Edge logs to verify startup
$ docker-compose logs -f edge
```

You should see logs like:
```
edge-1  | 2025-06-29T21:06:32.828371Z DEBUG unleash_edge::http::refresher::feature_refresher: Got updated client features. Updating features with Some(EntityTag { weak: false, tag: "537b2ba0:44" })
edge-1  | 2025-06-29T21:06:51.832151Z DEBUG unleash_edge::http::refresher::feature_refresher: No update needed. Will update last check time with "537b2ba0:44"
```

If your Edge node can't connect to the server, verify these common sources of problems. (Remember: if something is off, our [Slack community](https://www.getunleash.io/unleash-community) is glad to help!)

1. **Token is added correctly** - Double-check the token format and that it matches what you copied from the UI
2. **Edge node can reach Unleash server** - Test network connectivity:
   ```bash
   # Test if Edge can reach the Unleash server
   docker run --rm --network server_default curlimages/curl:latest curl -I http://web:4242/health
   ```

### Verify Feature Flag Sync

Visit `http://localhost:3063/internal-backstage/features` in your browser. You should see a JSON response containing your feature flags, confirming Edge is successfully fetching data from Unleash.

<img width="487" alt="image" src="https://github.com/user-attachments/assets/0db9a768-a7bd-40ae-97f6-04478ae26737" />

**NOTICE**: In production environment the `/internal-backstage/*` endpoints should not be publicly accessible!

### Switch Your Application

Update your SDK configuration to point to Edge instead of the Unleash server:

<img width="608" alt="image" src="https://github.com/user-attachments/assets/66362b9c-ebe6-4656-91a8-eb59ccfef095" />

```javascript
// NOTICE: your URI might be different, we are using a React-based frontend application
```

1. **Verify flag evaluation** by checking your application still receives feature flags correctly
2. **Test real-time updates** by toggling a feature flag in the Unleash UI (`http://localhost:4242`) and confirming your application reflects the change

### Test Resilience

Kill the Unleash server to verify Edge continues working:

```bash
# Stop the Unleash server
docker-compose kill web
```

Your application should continue to work as usual, as Edge serves cached feature flags.

<img width="591" alt="image" src="https://github.com/user-attachments/assets/8b73f6fc-0c03-4ff8-b394-8097da32c7e9" />

**You did a great job, it's done!**

## Next Steps

**Production Deployment**: Review the [Deployment Guide](https://docs.getunleash.io/reference/unleash-edge/deploying) for production configuration including TLS, load balancing, and scaling strategies.

**Advanced Configuration**: Explore Edge's [CLI options](https://docs.getunleash.io/reference/unleash-edge/cli) for custom headers, CORS settings, and backup strategies.

**Monitoring**: Set up observability with Edge's built-in metrics endpoint at `/internal-backstage/metrics` and configure logging levels for your environment.

**Security**: Configure proper API tokens, enable TLS, and review security best practices in the [official documentation](https://docs.getunleash.io/reference/unleash-edge).

**Migration**: If you're currently using Unleash Proxy, check out the [Migration Guide](https://docs.getunleash.io/reference/unleash-edge/migration-guide) for a smooth transition.

---

*Congratulations! You now have a resilient, high-performance feature flag setup that can handle serious traffic worldwide while keeping your applications running even during outages.*
