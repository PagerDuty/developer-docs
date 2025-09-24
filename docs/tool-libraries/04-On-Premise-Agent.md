---
tags: [tool-libraries]
---

# On Premise Agents

The PagerDuty Agent is a program that lets you easily integrate your monitoring system with PagerDuty. 
The officially supported agent is `pdagent` and the full documentation can be found [here](https://www.pagerduty.com/docs/guides/agent-install-guide/). 
A full list of PagerDuty agents is provided below.

<!-- theme:info -->
> ### Community Support
> PagerDuty does not endorse or provide technical support for any API libraries or tools created and maintained by our user community.

| Project Name | Language | Links | Integrates With | Supported By |
|:-------------|:---------|:-----------------|:-----------|:-------------|
| pdagent | Python | [Agent Install Guide](https://www.pagerduty.com/docs/guides/agent-install-guide/), [PagerDuty/pdagent](https://github.com/PagerDuty/pdagent) | Nagios, Sensu, Zabbix | PagerDuty |
| go-pdagent* | Golang | [PagerDuty/go-pdagent](https://github.com/PagerDuty/go-pdagent/) | | PagerDuty |
| go-pdagent | Golang | [m-masataka/go-pdagent](https://github.com/m-masataka/go-pdagent) | Zabbix | Community |
*This project originated inside of PagerDuty with the eventual goal of replacing the Python based `pdagent`.