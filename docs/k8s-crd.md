# CRDs
- each resource has a controller
- custom resource and custom controller

## CR
```
apiVersion: flights.com/vl
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

## CRD
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortnames:
    - ft
  versions:
  - name : v1
    served: true
    storage: true
  schema:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            from:
              type: string
            to:
              type: string
            number:
              type: integer
              minimum: 1
              maximum: 10
```

- CRDs allow you to create the resources and store them in ETCD
- you still need a controller that handles the creation and deletion of the CRs
- operator does what a human operator would do - manage, install, operate

Logs
Redirect logs to file in control plane
```
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/logs.txt
```
