---
title: 'Troubleshooting Network Outages: The Process I Use Before Making Changes'
description: 'A practical, evidence-based troubleshooting process for separating ISP, firewall, switching, wireless, DNS, and device-level network problems.'
pubDate: 'Oct 06 2026'
---

When somebody reports that “the network is down,” that statement can describe many different problems.

The entire location may have lost internet service. A secondary circuit may have failed without affecting users yet. One VLAN may be unavailable. Wireless clients may be disconnected while wired devices continue working. A single computer may have a bad address, or an application may be offline even though the network is healthy.

The fastest way to make an outage worse is to start changing configurations before understanding the scope. My approach is to slow the situation down just enough to gather useful evidence, then test from the outside in.

This article describes the troubleshooting process I use as a general framework. The examples are intentionally generic and do not contain real store information, internal addresses, or private configurations.

## First, define what “down” means

My first questions are simple:

- Who is affected?
- Which devices are affected?
- Is the problem wired, wireless, or both?
- Are internal resources unavailable, internet resources unavailable, or both?
- Did anything change shortly before the issue began?
- Is the outage complete, intermittent, or limited to one application?

These questions quickly separate a site-wide incident from a device problem.

If one workstation cannot reach an application while every other workstation is operating normally, I would not begin by rebooting the firewall. If wired and wireless devices across several networks all lose external connectivity at the same time, the investigation should move toward the firewall and internet circuits.

I also record the time the issue began. Accurate timing helps when reviewing logs and comparing the outage with provider events or configuration changes.

## Establish a known-good comparison

A working device is one of the best troubleshooting tools available.

If possible, I compare an affected device with a similar device that is still working. I look at its IP configuration, gateway, DNS servers, switch connection, wireless network, and application behaviour.

This helps answer whether the problem follows the user, device, cable, switch port, VLAN, or location. It also prevents me from treating a normal configuration value as suspicious just because I have not seen it before.

When there is no working device at the site, monitoring platforms and management dashboards become the comparison point. They can show whether infrastructure is reachable from the outside and when its status changed.

## Check the physical layer

Physical checks are basic, but they matter. Many outages come down to power, cabling, or an incorrect connection.

I verify:

- The modem, firewall, and switch have power.
- Network cables are connected to the expected ports.
- Link lights are present where expected.
- A cable has not been moved during construction or equipment changes.
- Power bars and UPS units are operating.
- The provider handoff is connected to the correct firewall interface.

I avoid repeatedly unplugging equipment without a reason. Every reboot removes useful state and can make timelines harder to understand. If I reboot something, I record what it was, when it was rebooted, and why.

## Validate the endpoint configuration

For an affected Windows device, I check the IP configuration before making changes. I want to know whether the device has a valid address, expected subnet mask, default gateway, and DNS servers.

An address beginning with `169.254` usually indicates the device did not receive a DHCP lease. A valid-looking IP address does not guarantee the device is in the correct VLAN, especially if a switch port was changed.

My basic test sequence is:

1. Ping the local loopback address.
2. Ping the device’s own address.
3. Ping the default gateway.
4. Test another internal destination if one should be reachable.
5. Test an external IP address.
6. Test a hostname.

Each result narrows the problem. If an external IP works but a hostname fails, DNS becomes the main suspect. If the gateway cannot be reached, there is little value in troubleshooting public DNS yet.

Ping is not always permitted, so I treat it as one piece of evidence rather than absolute proof. Port tests, route traces, application checks, and management telemetry may be required.

## Check switching and VLAN assignment

If the endpoint cannot reach its gateway, I look at the access layer.

I confirm that the switch is online, the port is active, and the port has the intended VLAN configuration. I also review whether the port is experiencing errors, frequent link changes, or an unexpected connected device.

MAC-address movement between ports can indicate a loop, an unmanaged switch, incorrect cabling, or a device that is legitimately changing paths. The event needs context before action is taken. Disabling ports immediately could disconnect more equipment without resolving the cause.

For wireless devices, I confirm the SSID, access-point association, signal quality, VLAN mapping, and whether the client received the expected address. A strong wireless signal does not mean the client has working upstream connectivity.

## Separate the firewall from the ISP

When a whole site loses external access, I try to determine whether the failure is on the local firewall, provider modem, circuit, or upstream service.

Useful questions include:

- Does the firewall show a physical link on the WAN interface?
- Does it have the expected WAN configuration?
- Can the firewall reach its next hop?
- Does the provider device show an alarm or loss of signal?
- Is the primary circuit down while the secondary circuit remains available?
- Did failover occur, and can essential traffic use the backup path?

If permitted and safe, connecting a test device directly to the provider handoff can help separate the ISP service from the firewall. That test must use the correct addressing information and should be performed carefully, because it bypasses normal network protections.

I would document the result precisely. “The modem works” is vague. “A directly connected test device reached an external IP using the assigned provider configuration” is useful evidence.

## Confirm routing, policy, and DNS

If interfaces are online but traffic still fails, I review the logical path.

The route table should contain an appropriate default route and any required internal routes. Firewall rules should allow the expected source, destination, and service. Network address translation should apply where required. A recent configuration change may explain why only one subnet or service is affected.

DNS deserves separate testing. I check whether the configured resolver is reachable and whether it can answer queries. If name resolution fails, users often describe the entire internet as down even though direct IP connectivity still works.

I avoid changing DNS servers permanently just because a public resolver works during a test. The organization may depend on internal DNS records or security controls that public DNS cannot provide. The test should identify the failure without creating a new design problem.

## Use logs and timelines

Management dashboards are helpful, but logs give the incident a timeline.

I look for WAN state changes, DHCP failures, authentication events, port changes, VPN status, configuration updates, and repeated connection attempts. I align those events with the time users first noticed the issue.

If a secondary connection had been down for several days before the primary failed, the visible outage may actually be the result of two separate failures. That is also a monitoring issue: the organization should have known the backup path was unavailable before it was needed.

## Make one controlled change at a time

Once I have a likely cause, I make the smallest reasonable change and test again.

Changing several settings, rebooting every device, and replacing cables simultaneously may restore service, but it makes the root cause unclear. It also makes the incident harder to prevent in the future.

For each action, I record:

- What I changed
- Why I changed it
- What result I expected
- What actually happened
- Whether I reversed the change

This turns troubleshooting into a repeatable investigation instead of trial and error.

## Validate recovery properly

An infrastructure dashboard returning to green is not enough. I verify service from the user’s point of view.

Depending on the incident, that may include testing a wired endpoint, wireless endpoint, business application, printer, voice device, and both internet connections. I also confirm that monitoring has cleared and that no temporary troubleshooting rule was left in place.

Finally, I document the root cause, impact, resolution, and follow-up actions. If the cause cannot be proven, I say that clearly and record the available evidence rather than presenting a guess as fact.

My overall process can be summarized as:

> Scope the issue, check the physical path, validate addressing, follow the traffic, review the evidence, make one controlled change, and test the real service.

That process is not complicated, but following it consistently makes network incidents faster to understand and much easier to explain afterward.
