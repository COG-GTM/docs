---
title: Healthchecks
description: Learn how to configure health checks to guarantee zero-downtime deployments of services on Railway.
---

Healthchecks guarantee zero-downtime [deployments](/deployments/reference) of your [service](/services) by ensuring the new version is live and able to handle requests before Railway routes traffic to it.

## How it works

When a new deployment is triggered for a service with a healthcheck endpoint configured, Railway keeps the previous deployment active and continues routing traffic to it. Railway queries the healthcheck endpoint on the new deployment until it receives an HTTP `200` response. Only then does Railway mark the new deployment as active, route traffic to it, and remove the previous deployment.

This process enables zero-downtime deployments: your users experience no interruption because the old version of the service continues to handle requests while the new version starts up. Traffic switches to the new deployment only after Railway confirms it is healthy.

If the new deployment fails to return a `200` status code within the [configured timeout](#healthcheck-timeout), Railway marks the deploy as failed and the previous version continues serving traffic.

**Note:** Railway does not monitor the healthcheck endpoint after the deployment has gone live. For continuous monitoring, see [continuous healthchecks](#continuous-healthchecks).

## Configure the healthcheck path

To configure a healthcheck:

1. Add a health endpoint to your application that returns an HTTP `200` status code when the service is ready to accept traffic. For example, a minimal Express endpoint:

    ```javascript
    app.get('/health', (req, res) => {
      res.status(200).json({ status: 'ok' });
    });
    ```

2. Under your service settings, input your health endpoint path (for example, `/health`). Railway queries this endpoint on each new deployment and routes traffic to it only after receiving a `200` response.

## Configure the healthcheck port

Railway will inject a `PORT` environment variable that your application should listen on.

This variable's value is also used when performing health checks on your deployments.

If your application doesn't listen on the `PORT` variable, possibly due to using [target ports](/networking/public-networking#target-ports), you can manually set a `PORT` [variable](/overview/the-basics#service-variables) to inform Railway of the port to use for health checks.

<Image
src="https://res.cloudinary.com/railway/image/upload/v1743469112/healthcheck-port_z0vj4o.png"
alt="Screenshot showing PORT service variable configuration"
layout="intrinsic"
width={1200} height={307} quality={100} />

Not listening on the `PORT` variable or omitting it when using target ports can result in your health check returning a `service unavailable` error.

## Healthcheck timeout

The default timeout on healthchecks is 300 seconds (5 minutes). If your application fails to serve a `200` status code during this allotted time, the deploy will be marked as failed.

<Image 
src="https://res.cloudinary.com/railway/image/upload/v1664564544/docs/healthcheck-timeout_lozkiv.png"
alt="Screenshot of Healthchecks Timeouts"
layout="intrinsic"
width={1188} height={348} quality={80} />

To increase the timeout, change the number of seconds on the service settings page, or with a `RAILWAY_HEALTHCHECK_TIMEOUT_SEC` service variable.

## Services with attached volumes

To prevent data corruption, we prevent multiple deployments from being active and mounted to the same service. This means that there will be a small amount of downtime when re-deploying a service that has a volume attached, even if there is a healthcheck endpoint configured.

## Healthcheck hostname

Railway uses the hostname `healthcheck.railway.app` when performing healthchecks on your service. This is the domain from which the healthcheck requests will originate.

For applications that restrict incoming traffic based on the hostname, you'll need to add `healthcheck.railway.app` to your list of allowed hosts. This ensures that your application will accept healthcheck requests from Railway.

If your application does not permit requests from that hostname, you may encounter errors during the healthcheck process, such as "failed with service unavailable" or "failed with status 400".

## Continuous healthchecks

The healthcheck endpoint is **_not used for continuous monitoring_**. Railway only calls it at the start of the deployment to ensure the service is healthy prior to routing traffic to it.

If you are looking for a way to set up continuous monitoring of your service(s), check out the <a href="https://railway.com/deploy/p6dsil" target="_blank">Uptime Kuma template</a> in the template marketplace.
