# When the protocol is the moat, look for the side door

**Trigger:** a target whose primary service is an industrial or specialty protocol — OPC-UA, Modbus, DNP3, BACnet, MQTT broker with cert auth, even some IIoT/SCADA stacks.

The protocol is dressed up for a reason. There's a published spec, mutual TLS, named cipher suites, message signing. Whoever built the box almost certainly leaned on those features as the front-door defense. The catch: the operator who configures the device, the technician who reads its dashboard, the engineer who downloads a CSV from it — none of them are speaking the protocol directly. They use a sidecar. Flask. Node-RED. Some PHP admin panel. That sidecar runs on a friendlier port, talks to the same underlying state, and was written by the same team in their non-specialist hand.

The flag, the credentials, the unfiltered command interface — they live on the sidecar far more often than on the protocol endpoint. Treat the specialty protocol as the loud distraction. Map the host. Look at every other open port. Whatever speaks HTTP and shares the box is the actual target.

The same principle applies to:

- Database servers fronted by a slim admin web UI on a high port (Redis Commander, pgAdmin, Mongo Express).
- Kerberos/AD environments where the real win is the SCCM console, not the KDC.
- Kubernetes clusters where you stop trying to break kube-apiserver and start poking the dashboard, Argo, Grafana, or the metrics-server.
- Hardware appliances whose configured purpose is one thing, but whose recovery web interface is another.

A heavily-defended specialty surface is a signal that the team's attention went there. Their other surfaces — held to a sidecar standard — are where the leverage lives.
