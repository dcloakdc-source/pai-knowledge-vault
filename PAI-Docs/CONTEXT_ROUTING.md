# Context Routing

Load context on-demand by reading the file at the path listed. Only load what the current task requires.

## Infrastructure & Assets (auto-loaded at session start)

| Topic | Path |
|-------|------|
| **Asset registry** (hosts, services, ports, paths) — **auto-loaded** | `PAI/ASSET_REGISTRY.md` |
| Full home inventory (hardware, network, topology) | `PAI/INFRA/HOME_INVENTORY.md` |
| Service interaction matrix (log pipelines, flows) | `PAI/INFRA/SERVICE_MATRIX.md` |
| Network topology diagram | `PAI/INFRA/TOPOLOGY.md` |
| Infrastructure design doc | `PAI/PAI_INFRASTRUCTURE_DESIGN.md` |
| System manifest (JSON) | `PAI/SystemManifest.json` |
| VLAN / network migration guide | `PAI/INFRA/MIGRATION_GUIDE.md` |

## PAI System

| Topic | Path |
|-------|------|
| PAI system overview | `PAI/README.md` |
| **Glossary / index** (all PAI acronyms, terms, codenames, concepts) | `PAI/DOCUMENTATION/GLOSSARY.md` |
| PAI harness overview (all layers) | `PAI/HARNESS.md` |
| System architecture | `PAI/PAISYSTEMARCHITECTURE.md` |
| **Architecture relationships** (Principal/DA/Engines/Agents/Archetypes/Personas/Hooks/Skills) | `PAI/ARCHITECTURE_RELATIONSHIPS.md` |
| Memory system | `PAI/MEMORYSYSTEM.md` |
| Skill system | `PAI/SKILLSYSTEM.md` |
| Skill consolidation policy | `PAI/config/skill-consolidation-manifest.example.yml`, `PAI/config/skill-consolidation-manifest.json`, `PAI/Tools/SkillConsolidationManifest.ts`, `PAI/Tools/SkillConsolidationProbe.ts` |
| Hook system | `PAI/THEHOOKSYSTEM.md` |
| Agent system | `PAI/PAIAGENTSYSTEM.md` |
| Org RBAC archetypes, personas, and engagement routing | `PAI/RBAC_ORG_STRUCTURES_MODEL.md`, `PAI/config/archetype-engagement.example.yml`, `PAI/Tools/validate-archetype-engagement.py` |
| Delegation system | `PAI/THEDELEGATIONSYSTEM.md` |
| Security system | `PAI/PAISECURITYSYSTEM/` |
| Notification system | `PAI/THENOTIFICATIONSYSTEM.md` |
| CLI architecture | `PAI/CLIFIRSTARCHITECTURE.md` |
| Tools reference | `PAI/TOOLS.md` |
| Token routing & local LLM | `MEMORY/STATE/token-routing-matrix.md` |
| Ollama A2A Bridge (port 8892) | `PAI/Tools/OllamaBridge/server.ts` — A2A agent, GET /agent-card, POST /execute |
| PAI Nova OpenCode (AGT-003) | `/PAI/Source/Projects/pai-opencode` — OpenCode v1.4.4, zen free, alias `opai`, Switchboard name `opencode` |
| Actions & pipelines | `PAI/ACTIONS.md`, `PAI/PIPELINES.md` |
| Flows | `PAI/FLOWS.md` |
| Behavioral rules | `PAI/AISTEERINGRULES.md` |
| PRD format spec | `PAI/PRDFORMAT.md` |

## User — Identity & Voice

| Topic | Path |
|-------|------|
| About User | `PAI/USER/ABOUTME.md` |
| Career & resume | `PAI/USER/RESUME.md` |
| Contacts | `PAI/USER/CONTACTS.md` |
| Personal rules | `PAI/USER/AISTEERINGRULES.md` |
| Opinions | `PAI/USER/OPINIONS.md` |
| Definitions | `PAI/USER/DEFINITIONS.md` |
| Core content themes | `PAI/USER/CORECONTENT.md` |
| Productivity system | `PAI/USER/PRODUCTIVITY.md` |
| Writing style | `PAI/USER/WRITINGSTYLE.md` |
| Rhetorical style | `PAI/USER/RHETORICALSTYLE.md` |

## User — Life Goals (Telos)

| Topic | Path |
|-------|------|
| Telos overview | `PAI/USER/TELOS/README.md` |
| Mission | `PAI/USER/TELOS/MISSION.md` |
| Goals | `PAI/USER/TELOS/GOALS.md` |
| Challenges | `PAI/USER/TELOS/CHALLENGES.md` |
| Beliefs | `PAI/USER/TELOS/BELIEFS.md` |
| Predictions | `PAI/USER/TELOS/PREDICTIONS.md` |
| Wisdom | `PAI/USER/TELOS/WISDOM.md` |
| Favorite books | `PAI/USER/TELOS/BOOKS.md` |
| Favorite movies | `PAI/USER/TELOS/MOVIES.md` |
| Favorite authors | `PAI/USER/TELOS/AUTHORS.md` |
| YouTube channels & creators | `PAI/USER/TELOS/CHANNELS.md` |
| Interest Audit (Core/Growth/Experiment) | `PAI/USER/TELOS/INTEREST_AUDIT.md` |
| Knowledge Ecosystem Map | `PAI/USER/TELOS/ECOSYSTEM_MAP.md` |

## DA (DA Identity)

| Topic | Path |
|-------|------|
| DA identity & rules | `PAI/USER/DAIDENTITY.md` |
| DA writing style | `PAI/USER/DAWRITINGSTYLE.md` |
| Our relationship | `PAI/USER/OUR_STORY.md` |

## User — Work

| Topic | Path |
|-------|------|
| Feed system | `PAI/USER/FEED.md` |
| Business context | `PAI/USER/BUSINESS/` |
| Health data | `PAI/USER/HEALTH/` |
| Financial context | `PAI/USER/FINANCES/` |
| All USER context index | `PAI/USER/README.md` |
