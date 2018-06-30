<a name="lightship"></a>
# Lightship

[![Travis build status](http://img.shields.io/travis/gajus/lightship/master.svg?style=flat-square)](https://travis-ci.org/gajus/lightship)
[![Coveralls](https://img.shields.io/coveralls/gajus/lightship.svg?style=flat-square)](https://coveralls.io/github/gajus/lightship)
[![NPM version](http://img.shields.io/npm/v/lightship.svg?style=flat-square)](https://www.npmjs.org/package/lightship)
[![Canonical Code Style](https://img.shields.io/badge/code%20style-canonical-blue.svg?style=flat-square)](https://github.com/gajus/canonical)
[![Twitter Follow](https://img.shields.io/twitter/follow/kuizinas.svg?style=social&label=Follow)](https://twitter.com/kuizinas)

Abstracts readiness/ liveness checks and graceful shutdown of Node.js services running in Kubernetes.

* [Lightship](#lightship)
    * [Behaviour](#lightship-behaviour)
        * [`/health`](#lightship-behaviour-health)
    * [Usage](#lightship-usage)
        * [Kubernetes container probe configuration](#lightship-usage-kubernetes-container-probe-configuration)
        * [Logging](#lightship-usage-logging)


<a name="lightship-behaviour"></a>
## Behaviour

Creates a HTTP service used to check [Container probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes).

Refer to the following Kubernetes documentation for information about the readiness and liveness checks:

* [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
* [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)

<a name="lightship-behaviour-health"></a>
### <code>/health</code>

`/health` endpoint describes the current state of a Node.js service.

The endpoint responds:

* `200` status code, message "SERVER_IS_READY" when server is accepting new connections.
* `500` status code, message "SERVER_IS_NOT_READY" when server is initialising.
* `500` status code, message "SERVER_IS_SHUTTING_DOWN" when server is shutting down.

<a name="lightship-usage"></a>
## Usage

Use `createLightship` to create an instance of Lightship.

```js
import {
  createLightship
} from 'lightship';

const configuration: LightshipConfigurationType = {
  onShutdown: () => {}
};

const lightship: LightshipType = createLightship(configuration);

```

The following types describe the configuration shape and the resulting Lightship instance interface.

```js
/**
 * @property port The port on which the Lightship service listens. This port must be different than your main service port, if any. The default port is 9000.
 * @property onShutdown A teardown function called to initialise the shutdown.
 * @property signals An a array of [signal events]{@link https://nodejs.org/api/process.html#process_signal_events}. Default: [SIGTERM].
 */
export type LightshipConfigurationType = {|
  +port?: number,
  +onShutdown: () => Promise<void> | void,
  +signals?: $ReadOnlyArray<string>
|};

/**
 * @property shutdown Changes server state to SERVER_IS_SHUTTING_DOWN and initialises the shutdown of the application.
 * @property signalNotReady Changes server state to SERVER_IS_NOT_READY.
 * @property signalReady Changes server state to SERVER_IS_READY.
 */
export type LightshipType = {|
  +shutdown: () => Promise<void>,
  +signalNotReady: () => void,
  +signalReady: () => void
|};

```

<a name="lightship-usage-kubernetes-container-probe-configuration"></a>
### Kubernetes container probe configuration

This is an example of a reasonable [container probe](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) configuration to use with Lightship.

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 9000
  periodSeconds: 5
  initialDelaySeconds: 5
livenessProbe:
  httpGet:
    path: /healthz
    port: 9000
  periodSeconds: 5
  initialDelaySeconds: 10

```

<a name="lightship-usage-logging"></a>
### Logging

`lightship` is using [Roarr](https://github.com/gajus/roarr) to implement logging.

Set `ROARR_LOG=true` environment variable to enable logging.