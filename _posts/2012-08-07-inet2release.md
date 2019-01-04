---
layout: post
title: INET 2.0 released
joomla_id: 3698
joomla_url: inet2release
date: 2012-08-07 00:00:00.000000000 +02:00
author: Administrator
excerpt: <div><div><span>We are happy to announce the INET Framework 2.0.0 release.
  The INET Framework is an open-source communication networks simulation package for
  the OMNeT++ simulation environment. It contains models for several wired and wireless
  networking protocols, including UDP, TCP, SCTP, IP, IPv6, Ethernet, PPP, 802.11,
  MPLS, OSPF, and many others. &nbsp;</span><span>We recommend that you port your
  existing models to the new 2.0 version so you can benefit from new features and
  improvements.</span></div><div><span><br /></span></div><div><span>V</span><span>isit
  the INET Framework website and</span><span>&nbsp;</span><a class="wikilink" href="http://inet.omnetpp.org/index.php?n=Main.Download"
  mce_href="http://inet.omnetpp.org/index.php?n=Main.Download">download it now</a><span>.
  Read on for details.</span></div></div><div><span>
category: Software
---
<div><div><span>We are happy to announce the INET Framework 2.0.0 release. The INET Framework is an open-source communication networks simulation package for the OMNeT++ simulation environment. It contains models for several wired and wireless networking protocols, including UDP, TCP, SCTP, IP, IPv6, Ethernet, PPP, 802.11, MPLS, OSPF, and many others. &nbsp;</span><span>We recommend that you port your existing models to the new 2.0 version so you can benefit from new features and improvements.</span></div><div><span><br /></span></div><div><span>V</span><span>isit the INET Framework website and</span><span>&nbsp;</span><a class="wikilink" href="http://inet.omnetpp.org/index.php?n=Main.Download" mce_href="http://inet.omnetpp.org/index.php?n=Main.Download">download it now</a><span>. Read on for details.</span></div></div><div><span></span></div><div><span><div><br /></div><div>New features:</div><div><br /></div><ul><li>INET is now partitioned into several "project features" that can be turned on or off independently. This can greatly reduce the compile time by turning off unused parts of INET.</li><li>Result recording has been ported to use the new signal-based statistics collection framework; this allows better separation of the model and the statistic collection code.</li><li>New Differentiated Services framework for QoS simulations.</li><li>New IPv4NetworkConfigurator for more powerful configuration of IP networks.</li><li>New protocols: DHCP, BGPv4; intergated xMIPv6 (mobile IPv6).</li><li>New MANET routing protocols (from INETMANET): AODV, DSR, BATMAN, DYMO, OLSR</li><li>New PcapRecorder module for capturing traffic traces.</li><li>New VoIP application that allows sending an actual voice stream over the network.</li><li>Integrated HttpTools for simulating HTTP-based applications.</li><li>New mobility modules including TraCI (taken over from the Veins project)</li><li>Added an LwIP-based TCP implementation.</li><li>Writing a manual for INET has been started and already made a great progress.</li></ul><div><br /></div><div>Improvements:</div><ul><li>Node models refactored for better extensibility (StandardHost, AdhocHost, WirelessHost, Router).</li><ul><li>Different types of TCP, UDP and SCTP applications can co-exist in the same host.</li><li>Alternate UDP and TCP implementation can be plugged-in. Three independent TCP implementations are available (OMNeT++ native, LwIP, NSC)</li><li>Inside a host, submodules are instantiated only if they are actually required.</li><li>Configurable hooks have been added in the network layer to allow packet drop/duplication scenarios.</li><li>Routers now support an unlimited number of wireless, Ethernet, point-to-point and external interfaces.</li><li>Mobility support in Router and StandardHost is now optional.</li><li>AccessPoints now have both wireless and Ethernet interfaces (bridged).</li></ul><li>Revised OSPFv2 model.</li><li>TCP: Transmission mode (byte count, object and byte stream) is now specified by the application.</li><li>UDP: multicast, broadcast and TTL support. Improved socket API.</li><li>IPv6: implemented default router selection, tunneling and datagram fragmentation/reassembly for PPP links.</li><li>IPv4: reimplemented multicast routing</li><li><span>Ethernet:&nbsp;EtherMAC refactored for better readability; added reconnect support, better PAUSE support, support for 40 and 100 Gigabit Ethernet.</span></li><li>Ethernet datarate is now configured on the channels (not in the MAC). Also added new Ethernet channel types: Eth10M, Eth100M, Eth1G, Eth10G, Eth40G and Eth100G</li><li>IEEE802.11 a/b/g/s model: Unified several implementations into a single MAC module.</li><li>Multiple radio support for wireless hosts; radio infrastructure has been refactored.</li><li>Refactoring: Mobility support is now completely independent of the radio infrastructure and it is compatible with the MiXiM mobility modules.</li><li>A comprehensive test suite has be devised, and deployed in a continuous testing environment (Jenkins).</li><li>A large number of bug fixes and other improvements.</li></ul><div><span><br /></span></div><div><span>You can find the detailed change log&nbsp;</span><a class="urllink" href="https://github.com/inet-framework/inet/blob/master/WHATSNEW" mce_href="https://github.com/inet-framework/inet/blob/master/WHATSNEW" rel="nofollow">here</a>.</div><p><br /></p></span></div>