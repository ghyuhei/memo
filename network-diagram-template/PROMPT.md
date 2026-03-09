# Network Diagram Generator — Prompt for Claude Code

> **This file is the generation instruction.**
> When a user asks Claude Code to generate a draw.io diagram from an input YAML,
> Claude Code should follow these rules.

## How to use

```
Claude Code, please read PROMPT.md and generate output.drawio from input.yaml
```

---

## Generation Rules

### 1. Read input YAML and validate

- All `???` placeholders must be replaced before generation (warn the user if found)
- CIDRs should not overlap across VPCs/VNets
- Each `id` field must be unique within its section
- Tabs are auto-determined: generate only tabs for which input data exists

### 2. Tab structure

| Tab | ID | Name | When to generate |
|-----|----|------|------------------|
| 1 | `L1-overview` | L1 Overview | Always |
| 2 | `L2-{center}` | L2 {center} Detail | Always (center = `diagram.center`) |
| 3 | `L2-other-clouds` | L2 Other Clouds | If any non-center cloud section exists (azure, alibaba, etc.) |
| 4 | `L2-dc-overseas` | L2 DC & Overseas | If `dc` or `overseas` section exists |

- Tab 1: Hub-and-spoke topology. `diagram.center` at center. All other environments around it. Connection lines labeled with type and bandwidth.
- Tab 2: Detailed view of the center environment. VPCs → Subnets → Services. Regional services outside VPCs. External connections as labeled stubs.
- Tab 3: Detail for non-center cloud environments. Each gets its own section on the same page.
- Tab 4: Detail for on-premises DCs and overseas subsidiaries.

### 3. Page Links (drill-down navigation)

- **L1 Overview**: Each environment box is a clickable `UserObject` with `link="data:page/id,{TAB_ID}"`
- **Detail tabs**: Include "← Back to L1 Overview" link at top-left using `UserObject` with `link="data:page/id,L1-overview"`

### 4. Visual Conventions

#### Colors by environment
| Environment | Stroke Color | Fill Color (light) |
|-------------|-------------|-------------------|
| AWS | `#FF9900` | `#FFF8EF` |
| Azure | `#0078D4` | `#E8F4FD` |
| Alibaba | `#FF6A00` | `#FFF3E8` |
| DC (on-prem) | `#6C757D` | `#F1F1F1` |
| Overseas | `#28A745` | `#E8F5E9` |

#### Colors by service category (icon fill)
| Category | Fill Color | Examples |
|----------|-----------|----------|
| networking | `#8C4FFF` | ALB, NAT GW, Route 53, TGW, CloudFront, SLB |
| compute | `#ED7100` | EC2, ECS, EKS, Lambda, App Service, AKS |
| database | `#C925D1` | RDS, Aurora, ElastiCache, SQL Database, DynamoDB |
| storage | `#3F8624` | S3, EFS, OSS |
| security | `#DD344C` | WAF, GuardDuty, IAM, Firewall |
| management | `#E7157B` | CloudWatch, CloudTrail, Config |

#### Subnet border colors
| Subnet type | Stroke Color | Fill Color |
|------------|-------------|-----------|
| public | `#7AA116` | `#F2F6E8` |
| private | `#00A4A6` | `#E6F6F7` |
| db | `#DD344C` | `#FBE5E7` |

#### DC zone colors
| Zone pattern | Stroke Color | Fill Color |
|-------------|-------------|-----------|
| DMZ | `#DD344C` | `#FBE5E7` |
| Internal | `#6C757D` | `#F1F1F1` |
| Cloud Connect | `#FF9900` | `#FFF8EF` |

#### Connection line styles
| Connection type | Style | Color | Width |
|----------------|-------|-------|-------|
| `direct_connect` | Solid | `#FF9900` | 3px |
| `peering` | Solid | Target environment color | 3px |
| `vpn` | Dashed | `#28A745` | 2px |
| `sd-wan` | Dashed | `#17A2B8` | 2px |
| `expressroute` | Solid | `#0078D4` | 3px |
| `cen` | Solid | `#FF6A00` | 3px |
| `private_wan` (DC-DC) | Solid | `#6C757D` | 2px |
| Internal traffic | Solid | `#232F3E` | 1px |

### 5. AWS services — use official AWS4 icon shapes

Use `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.{icon_name}` for AWS services.

Common icon mappings:
```
ALB / ELB           → elastic_load_balancing
EC2                 → ec2
ECS                 → ecs
EKS                 → eks
Lambda              → lambda
RDS                 → rds
Aurora              → aurora
DynamoDB            → dynamodb
ElastiCache         → elasticache
S3                  → s3
Route 53            → route_53
CloudWatch          → cloudwatch
IAM                 → identity_and_access_management
GuardDuty           → guardduty
WAF                 → waf
CloudFront          → cloudfront
NAT Gateway         → nat_gateway
Internet Gateway    → internet_gateway
Transit Gateway     → transit_gateway
Direct Connect      → direct_connect
VPN                 → site_to_site_vpn
```

For Azure and Alibaba services, use rounded rectangles colored by `category`.

### 6. Layout rules

**L1 Overview** (page size: 1600 x 1200):
- Center environment: middle of canvas (~x:500, y:350, w:600, h:300)
- Other clouds: top-left and top-right
- DCs: left side, stacked vertically
- Overseas: right side, stacked vertically
- Internet cloud: top center (use ellipse shape, not `group_on_premise`)
- Legend: bottom center

**L2 Detail pages** (page size: 2200 x 1700):
- Main region/environment: large box occupying most of the canvas
- External connections: small labeled stub boxes at edges with arrows
- Gateway components (TGW, DXGW, IGW): top of the region box
- VPCs/VNets: arranged within region, top-to-bottom or left-to-right
- Subnets: inside VPCs, arranged left-to-right
- Services: as icons (AWS) or colored boxes (non-AWS) inside subnets
- Regional services (S3, WAF, IAM, GuardDuty): inside region but OUTSIDE any VPC

### 7. Architectural correctness

- **S3, CloudFront, WAF, DynamoDB** are regional services → place OUTSIDE any VPC
- **IAM, GuardDuty, CloudTrail** are global/regional services → place OUTSIDE any VPC
- **Route 53 Resolver** has VPC endpoints → can be shown inside a VPC
- **CloudWatch** is accessible via VPC endpoints → can be shown inside a VPC
- **Internet Gateway** is at VPC level → attach to VPCs with public subnets
- **NAT Gateway** is inside a public subnet
- **ALB** belongs in a public subnet (internet-facing) or private subnet (internal)
- **Direct Connect path**: DC Router → Direct Connect GW → Transit GW → VPCs
- **VPN path**: Overseas VPN → Transit GW → VPCs
- **Peering path**: Other Cloud → Transit GW (or VPC Peering) → VPCs
- **GuardDuty and WAF are separate services** — never combine in one icon

### 8. Legend

Every tab must include a legend box at the bottom:
- L1: Connection line styles (as actual short edge lines) + environment color key
- L2: Subnet type colors + service category colors

### 9. Notes and annotations

- `cross_cutting` section: render as a note box on L1 Overview (bottom or side)
- `notes` fields: render as italic text near the relevant component
- Compliance notes (GDPR, ICP, etc.): render in red italic text

### 10. Input YAML field reference

Key field names (used across all environments):
- `connection`: how this environment connects to the center (replaces any `connection_to_aws` from older templates)
- `connection.type`: connection type (matches line style table above)
- `connection.via`: intermediary if applicable (e.g., "Megaport")
- `connection.bandwidth`: bandwidth label for the connection line
- `regional_services`: services outside any VPC/VNet, at the region level
- `dc.sites[].connection`: optional — omit entirely if the site has no direct connection to center
