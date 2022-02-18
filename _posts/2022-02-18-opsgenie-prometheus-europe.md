---
layout: post
title: Configuring Opsgenie in the EU
subtitle: or how I almost cried because I was using the wrong API
categories: monitoring
tags: [kubernetes, opsgenie, monitoring, prometheus]
---

Hey guys today is more a short rant than anything but my goal was to configure opsgenie for a prometheus deployment through alertmanager.

All that sounds easy and quick to do especially when you red tutorials like:

<https://kb.vshn.ch/vshnsyn/how-tos/opsgenie.html>

Or look on github

<https://github.com/prometheus/alertmanager/issues/1565>

or even their own documentation

<https://prometheus.io/docs/alerting/latest/configuration/#opsgenie-receiver->

In General the entire operation is very simple:

- get your API key from the dashboard under teams -> your team -> integrations 
- get your team id: the simplest way i have found is to simply copy it from the querystring "?teamId=" of your dashboard when you create your integration
- paste both into the alert manager config  and adapt your configs and done.

so in the end it will looks something like this:

``` Alertmanager.Yml
global:
  resolve_timeout: 1m
  opsgenie_api_key: "<apikey>"
route:
  receiver: "opsgenie"
  group_wait: 10s
  group_by: [alertname]
  repeat_interval: 24h
  routes:
    - receiver: "opsgenie"
      matchers:
        - severity=~"major"
    - receiver: "opsgenie"
      matchers:
        - severity=~"critical"
        - production="false"

receivers:
  - name: "opsgenie"
    opsgenie_configs:
      - priority: '{{ if eq .GroupLabels.severity "critical" }}P1{{ else if eq .GroupLabels.severity "warning" }}P2{{ else if eq .GroupLabels.severity "info" }}P3{{ else }}P4{{ end }}'
        message: '[{{ .CommonLabels.tenant_id }}/{{ .CommonLabels.cluster_id }}] {{ .GroupLabels.alertname }} in {{ .GroupLabels.namespace }}'
        description: |-
          {{ if gt (len .Alerts.Firing) 0 -}}
          Alerts Firing:
          {{ range .Alerts.Firing }}
            - Message: {{ .Annotations.message }}
              Labels:
          {{ range .Labels.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Annotations:
          {{ range .Annotations.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Source: {{ .GeneratorURL }}
          {{ end }}
          {{- end }}
          {{ if gt (len .Alerts.Resolved) 0 -}}
          Alerts Resolved:
          {{ range .Alerts.Resolved }}
            - Message: {{ .Annotations.message }}
              Labels:
          {{ range .Labels.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Annotations:
          {{ range .Annotations.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Source: {{ .GeneratorURL }}
          {{ end }}
          {{- end }}
        details:
          namespace: '{{- if .CommonLabels.exported_namespace -}}{{- .CommonLabels.exported_namespace -}}{{- else if .CommonLabels.namespace -}}{{- .CommonLabels.namespace -}}{{- end -}}'
          pod: '{{- if .CommonLabels.pod -}}{{- .CommonLabels.pod -}}{{- end -}}'
          deployment: '{{- if .CommonLabels.deployment -}}{{- .CommonLabels.deployment -}}{{- end -}}'
          alertname: '{{ .GroupLabels.alertname }}'
          cluster_id: '{{ .CommonLabels.cluster_id }}'
          tenant_id: '{{ .CommonLabels.tenant_id }}'
          severity: '{{ .GroupLabels.severity }}'
        tags: '{{ .CommonLabels.tenant_id }},
          {{ .CommonLabels.cluster_id }},
          {{ .GroupLabels.severity }},
          {{ .GroupLabels.alertname }},
          {{ .GroupLabels.namespace }},
          {{- if .CommonLabels.exported_namespace -}}{{ .CommonLabels.exported_namespace }},{{- end -}}'
        responders:
          - id: "<teamid>"
            type: "team"
```

Now if you are not from the eu you are done. But if you are you will get a greeting like this in the alertmanager pod logs:

``` Pod 
level=error ts=2022-02-17T17:03:19.184Z caller=dispatch.go:354 component=dispatcher msg="Notify for alerts failed" num_alerts=1 err="opsgenie/opsgenie[0]: notify retry canceled due to unrecoverable error after 1 attempts: unexpected status code 401: {\"message\":\"Could not authenticate\",\"took\":0.0,\"requestId\":\"d8bbd893-8eaa-4e1d-be4d-282eb38f6629\"}"
```

And you will be sitting their going "huh, i thought opsgenie is integrated into alert manager and should therefore just work???" or "I literally only have two auth fields, the API_KEY and the TEAM_ID and I copied both from their dashboard. So whats going on?

Curling the API yourself is an option. So lets do that:

``` Console
$ curl -H "Authorization: GenieKey $key" "https://api.opsgenie.com/v2/teams/$teamname?identifierType=name" | jq -r '.data.id'
```

but that will yield the same result "Could not authenticate". So if you are like me you will assume it has to be my fault. So I rewrote the config in different ways, with different styles with different indentations and different quotes. But the system was having none of it.

So I went to bed and meditated over it at night. Until it the next morning, with the help of some coffee I saw it in the address bar of opsgenie:

"https://MyApp.app.eu.opsgenie.com/" do you see it yet?

The client I was helping with is an EU customer and therefore Atlassian would put them on european servers, which to my surprise, don't talk to each other.

All you need to do is change the Opsgenie API url to european servers: 

```
 opsgenie_api_url: "https://api.eu.opsgenie.com/" 
```

And you would be up and running. Making our final config look like this:

``` Alertmanager.Yml
global:
  resolve_timeout: 1m
  opsgenie_api_key: "<apikey>"
  opsgenie_api_url: "https://api.eu.opsgenie.com/" 
route:
  receiver: "opsgenie"
  group_wait: 10s
  group_by: [alertname]
  repeat_interval: 24h
  routes:
    - receiver: "opsgenie"
      matchers:
        - severity=~"major"
    - receiver: "opsgenie"
      matchers:
        - severity=~"critical"
        - production="false"

receivers:
  - name: "opsgenie"
    opsgenie_configs:
      - priority: '{{ if eq .GroupLabels.severity "critical" }}P1{{ else if eq .GroupLabels.severity "warning" }}P2{{ else if eq .GroupLabels.severity "info" }}P3{{ else }}P4{{ end }}'
        message: '[{{ .CommonLabels.tenant_id }}/{{ .CommonLabels.cluster_id }}] {{ .GroupLabels.alertname }} in {{ .GroupLabels.namespace }}'
        description: |-
          {{ if gt (len .Alerts.Firing) 0 -}}
          Alerts Firing:
          {{ range .Alerts.Firing }}
            - Message: {{ .Annotations.message }}
              Labels:
          {{ range .Labels.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Annotations:
          {{ range .Annotations.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Source: {{ .GeneratorURL }}
          {{ end }}
          {{- end }}
          {{ if gt (len .Alerts.Resolved) 0 -}}
          Alerts Resolved:
          {{ range .Alerts.Resolved }}
            - Message: {{ .Annotations.message }}
              Labels:
          {{ range .Labels.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Annotations:
          {{ range .Annotations.SortedPairs }}   - {{ .Name }} = {{ .Value }}
          {{ end }}   Source: {{ .GeneratorURL }}
          {{ end }}
          {{- end }}
        details:
          namespace: '{{- if .CommonLabels.exported_namespace -}}{{- .CommonLabels.exported_namespace -}}{{- else if .CommonLabels.namespace -}}{{- .CommonLabels.namespace -}}{{- end -}}'
          pod: '{{- if .CommonLabels.pod -}}{{- .CommonLabels.pod -}}{{- end -}}'
          deployment: '{{- if .CommonLabels.deployment -}}{{- .CommonLabels.deployment -}}{{- end -}}'
          alertname: '{{ .GroupLabels.alertname }}'
          cluster_id: '{{ .CommonLabels.cluster_id }}'
          tenant_id: '{{ .CommonLabels.tenant_id }}'
          severity: '{{ .GroupLabels.severity }}'
        tags: '{{ .CommonLabels.tenant_id }},
          {{ .CommonLabels.cluster_id }},
          {{ .GroupLabels.severity }},
          {{ .GroupLabels.alertname }},
          {{ .GroupLabels.namespace }},
          {{- if .CommonLabels.exported_namespace -}}{{ .CommonLabels.exported_namespace }},{{- end -}}'
        responders:
          - id: "<teamid>"
            type: "team"
```

And Boom, you are in business. You will notice that pretty quickly because everybody in your team that is on the on call list will no receive live emails from opsgenie.

So thats it. 
I hope that this way at least some of you will be spared my debugging time.
