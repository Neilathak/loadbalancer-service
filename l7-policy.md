---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-12"

keywords: l7, layer 7, policy, policies

subcollection: loadbalancer-service

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:note: .note}
{:important: .important}
{:tip: .tip}

# Layer 7 Policy
{: #layer-7-policy}

A Layer 7 (L7) policy is used to classify traffic by matching its L7 information with L7 rules, and then taking specific actions if those rules match.

* A policy is applied to a front-end application port (protocol).
* Multiple policies can be applied to the same protocol.

Since multiple policies can be applied to a protocol, there is a priority associated with each policy.

* Policies with the lowest set priority are evaluated first.
* If the rules associated with the policy do not match the traffic, the next lowest policy on the priority list is evaluated.

If the traffic does not match any of the policy rules, the traffic is redirected to a default pool, which is the pool that was configured when the basic load balancer was deployed.

Each policy is associated with an action that is performed when all of the rules in the policy match the traffic.

The actions can be:

- Reject
- Redirect to a URL
- Redirect to pool

Policies set to `reject` are evaluated first, regardless of their priority.

After that, policies set to `redirect to url` are evaluated.

Finally, policies set to `redirect to pool` are evaluated.

Within each action category, the policies are evaluated in ascending order of priority (lowest to highest).

## Layer 7 Policy Properties

Property  | Description
------------- | -------------
Name | The name of the policy. Each policy must have a unique name.
Action | The action to take when the rules match. The actions are `REJECT`, `REDIRECT_URL`, and `REDIRECT_POOL`. A policy whose action is set to `REJECT` is always evaluated first, regardless of its priority. Policies whose actions are set to `REDIRECT_URL` are evaluated next, followed by policies whose actions are set to `REDIRECT_POOL`.
Priority | Policies are evaluated based on ascending order of priority.
Redirect URL | The URL to which traffic will be re-directed, if the action is set to `REDIRECT_URL`.
Redirect L7 Pool | The pool of servers to which traffic will be sent, if the action is set to `REDIRECT_POOL`.
Protocol | The front-end application port to which the policy is applied.

## Layer 7 Rule
Layer 7 rules define a portion of the incoming traffic that is to be matched with specific values.

* If the incoming traffic matches the specified value of a rule, then the rule evaluates to `true`.
* Layer 7 rules are always associated with a Layer 7 policy. Multiple Layer 7 rules can be associated with the same layer 7 policy.
* If multiple rules are associated with a policy, then each rule will be evaluated to be `true` or `false`.
* If all the rules associated to a policy evaluate to `true`, then the policy action will be applied to the request. Otherwise, the load balancer evaluates the next policy.

Rules have types, which can be:

* `HOST_NAME`
* `FILE_TYPE`
* `HEADER`
* `COOKIE`
* `PATH`

These indicate the portion of the Layer 7 traffic to be matched with the rule.

Type      |  Field to be extracted and evaluated
----------| -----------------------
`HOST_NAME` | The hostname part of the URL (for example, `api.my_company.com`)
`FILE_TYPE` | The end of the URL, representing the file type (for example, `jpg`)
HEADER    | A field in the HTTP header
COOKIE    | A named cookie in the HTTP header
PATH      | The part of the URL that follows the hostname (for example, `/index.html`)

Rules also have a comparison type, which indicates how they are are to be evaluated. 
Rules can have following comparison types:

* `REGEX`
* `STARTS_WITH`
* `ENDS_WITH`
* `CONTAINS`
* `EQUAL_TO`

Comparison Type |  Type of evaluation
----------------|---------------------
`REGEX`           |  Match the extracted field (for example, `hostname`) with the supplied regular expression
`STARTS_WITH`     |  Verify whether the extracted field starts with the supplied string
`ENDS_WITH`       |  Verify whether the extracted field ends with the supplied string
`CONTAINS`        |  Verify whether the extracted field contains the supplied string
`EQUAL_TO`        |  Verify whether the extracted field is identical to the supplied string

Not all rule types support all comparison types. For example, if you are using `FILE_TYPE`, it is best to use comparison types `REGEX` and `ENDS_WITH`.
{: tip}

## Layer 7 Rule Properties

Property  | Description
------------- | -------------
Type | Specifies the type of rule. Rule types can be `HOST_NAME` (the name of the host), `FILE_TYPE` (the type of the file), `HEADER` (the header), `COOKIE` (the cookie) or `PATH` (the path).
Comparison Type | Comparison types are used in association with the rule type, key and value to define a rule and classify traffic. Comparison types can be: `REGEX`, `STARTS_WITH`, `ENDS_WITH`, `CONTAINS`, and `EQUAL_TO`.
Key | The description key for the rule types `HEADER` and `COOKIE`.
Value |  For the rule types `HEADER` and `COOKIE`, the value is compared against the key.
Invert | It you set the value to 1, the value of this L7 rule comparison is set to `true` whenever the specified rule is not matched.
Layer 7 Policy ID | The unique identifier of the policy to which the rules are attached.
