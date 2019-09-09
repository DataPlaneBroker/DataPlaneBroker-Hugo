---
title: 'DataPlane Broker Introduction'
date: 2019-09-01T19:27:37+10:00
weight: 1
---
## NFV-MANO data model and the WIM

NFV management standardization is predominantly driven by the [ETSI NFV SIG](https://www.etsi.org/technologies/nfv). The group develops a wide portfolio of standards and reference implementations, including the NFV-MANO architecture and data models. The proposed architecture consists of three management layers: 

 * The Virtual Infrastructure Manager (VIM) controlling physical infrastructure resources; 
 * The NFV manager (NFVM) controlling and monitoring running NFV instances; 
 * The NFV Orchestrator (NFVO) responsible for fulfilling network service requests from external users.

Although NFV-MANO was initially designed to support NFV management within a single datacenter, the development of new technological paradigms, like Mobile Edge Cloud and cloud-RAN, have introduced a requirement in standardization efforts to develop support for cross-site service deployment. Developing relevant support is hindered by the lack of support for management of network  resources and  forwarding in the existing data model. The existing NFV-MANO specification offset these control and management capabilities to a WAN Infrastructure Manager (WIM)~\cite{sgi-etsi-wim} entity and declare out-of-scope the specification of the interface between the NVFO and the WIM. A WIM is a special type of a VIM which offers exclusively control of network resources and connectivity between NFV datacenters. In effect, it is a shim layer mediating MANO virtual link requests to network controllers, in an effort to provide point-to-point and multi-point connectivity between NFVs of a single network service across datacenters.

## DataPlane Broker

## DPB Model

In order to support the technology and topology diversity, DPB develops a hierarchical model to abstract network connectivity support between the different network management layers. The DPB NBI follows a *one big switch* abstraction; the underlying topology remains hidden from the orchestrator, which can request connectivity for any arbitrary group of network terminals, which are points of access to VIMs' internal networks. Through the DPB, the orchestrator can manage **a (logical) network**, a dynamic set of **network services**, i.e. layer-2 broadcast domains, between network terminals and with specific bandwidth guarantees. A network service is modeled as a set of _circuit-id, ingress-bandwidth, egress-bandwidth_ tuples. A circuit is an endpoint of a service, identified by the terminal it resides on and a numeric _label_ (e.g. a VLAN id) distinguishing it from others services occupying the same terminal.  The following picture depicts a logical network with six terminals, and one service connecting three circuits (site1-opst:435, site2-ofx:961, site3-ofx:2010) and another service connecting two (site1-opst:91, site1-ofx:961).

<center>
![Sample DPB service](../../images/network-abstr.png)
</center>

The DPB defines two logical network types. A __logical switch network__ is a network of the simplest type, modeling a single switching fabric, e.g. an OpenFlow datapath. An __aggregator__ is a complex network abstracting the control and connectivity of different inferior networks, which can be either switch networks or smaller aggregator networks. Physical connections between a subset of inferior networks' terminals are modeled as _trunks_, each comprised of bandwidth capacity and available circuit labels. Management and control of each trunk is the responsibility of exactly one aggregator. An aggregator's own terminals are mapped to inferiors' terminals that are not internally connected by its trunks.

