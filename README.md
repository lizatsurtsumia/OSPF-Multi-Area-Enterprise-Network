# OSPF-Multi-Area-Enterprise-Network


A hands-on OSPF lab built in EVE-NG/GNS3 with 10 routers, 2 switches and 1 host. It covers area design, special area types, route summarization, authentication, LSA analysis and adjacency troubleshooting.

Topology

Topology

Area	Routers	Type	Role
Area 0	R1, R2, R3, R4 (20.20.20.0/24), R7-R8 (20.20.30.0/24)	Backbone	R7-R8 reaches the backbone through a virtual link
Area 51	R2, R4 (ABRs), R5	NSSA	R5 is an ASBR toward R6
Area 2	R3, R7 (ABRs), H1	Normal	Transit area for the R3-R7 virtual link
Area 3	R3 (ABR), R9, R10	Totally stubby	No ASBR, default route only
External	R6	EIGRP (redistributed on R5)	Loopbacks 172.16.10-12.0/24
Internet	R1	Static default, default-information originate	203.0.113.0/30

Addressing
Links
Subnet	Devices	Area
203.0.113.0/30	R1 .1, Internet .2	n/a
20.20.20.0/24 (S11)	R1 .1, R2 .2, R3 .3, R4 .4	0
192.168.1.0/24	R4 .4, R5 .5	51
192.168.2.0/24	R2 .2, R5 .5	51
192.168.20.0/24	R5 .5, R6 .6	EIGRP
192.168.6.0/24	R3 .3, R7 .7	2
192.168.5.0/24	R7 .7, H1 .10	2
20.20.30.0/24	R7 .7, R8 .8	0
192.168.3.0/24 (S12)	R3 .3, R9 .9, R10 .10	3
Loopbacks
Router	Loopback	Area
R1	1.1.1.1/32	0
R2	2.2.2.2/32	0
R3	3.3.3.3/32	0
R4	4.4.4.4/32	0
R5	5.5.5.5/32	51
R6	6.6.6.6/32, 172.16.10.6/24, 172.16.11.6/24, 172.16.12.6/24	EIGRP
R7	7.7.7.7/32	2
R8	8.8.8.8/32	0
R9	9.9.9.9/32	3
R10	10.10.10.10/32	3

Full details are in topology/addressing.md.

Lab phases
Phase	Folder	What it covers
1	configs/01-baseline	Multi-area OSPF, virtual link R3-R7, default route from R1, EIGRP redistribution on R5
2	configs/02-stub-nssa	Area 3 totally stubby, Area 51 NSSA
3	configs/03-summarization	Inter-area area range and external summary-address
4	configs/04-authentication	MD5 per area plus a key on the virtual link
5	05-lsa-analysis	LSA types 1-7 captured and explained
6	06-troubleshooting	Adjacency failures: symptom, diagnosis, fix
Key design decisions
Area 2 stays a normal area. Virtual links cannot cross stub or NSSA areas, and the R3-R7 virtual link transits Area 2.
Area 51 is an NSSA because R5 redistributes R6's routes (ASBR). A plain stub area cannot contain an ASBR.
Area 3 is totally stubby (area 3 stub no-summary on R3) because it has no ASBR. R9 and R10 only get a default route.
Area 0 is discontiguous (R7-R8). The virtual link between R3 (3.3.3.3) and R7 (7.7.7.7) through Area 2 fixes this.
Results

Fill these in with your own outputs.

Check	Expected	Evidence
All adjacencies FULL	show ip ospf neighbor	outputs/
Virtual link FULL	show ip ospf virtual-links on R3/R7	
R9 routing table	Only O*IA 0.0.0.0/0 plus intra-area	
R5 routing table	O*IA 0.0.0.0/0 via R2/R4	
R1 sees R6 loopbacks	O E2 172.16.x.x (type 7 translated to type 5)	
Summaries	One /22 for 192.168.0-3.x, one /21 for 172.16.8-15.x	
Authentication	Neighbors FULL with MD5, down with a wrong key	
LSA analysis

See 05-lsa-analysis/analysis.md. Database output was captured on R9 (stub), R5 (NSSA), R1 (backbone) and R7 (transit), with Wireshark captures of type 7 translation.

Troubleshooting log
#	Problem	Root cause	Write-up
1	R1 had no neighbor with R2	R2 was not cabled to the 20.20.20.0/24 switch	01
2	R9 had a neighbor with R10 only, no default route	Stub flag mismatch with R3	02
3+	Induced failures: MTU, auth, timers, area ID, passive interface	See each file	06-troubleshooting
Verification commands
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip ospf database
show ip ospf database database-summary
show ip ospf border-routers
show ip ospf virtual-links
debug ip ospf adj


