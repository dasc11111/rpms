[README.md](https://github.com/user-attachments/files/30108483/README.md)
# RPMS — Radiation Protection Management System

> Plataforma SaaS multi-tenant, orientada a IA, para la **gestión integral del ciclo de vida de la protección radiológica** en instalaciones médicas, industriales, académicas y regulatorias de Latinoamérica.

**Versión actual:** `v0.1.0-mvp-foundations` (Fundaciones del MVP)
**Estado:** Fundaciones técnicas y de dominio listas.

---

## 📋 En una línea

Reemplaza el modelo tradicional **"carpeta compartida + Excel + memoria del OPR"** por un ecosistema donde la información se ingresa una sola vez, se comprende automáticamente y se relaciona sola — con IA en el núcleo, no como capa cosmética.

---

## 🎯 Principios de producto

| # | Principio | Traducción operacional |
|---|-----------|------------------------|
| P1 | Regla de los 3 clics | Información crítica accesible en ≤3 clics desde el dashboard |
| P2 | Cero digitación redundante | El dato se captura una vez y se propaga a todas las entidades |
| P3 | IA proactiva, no reactiva | El sistema anticipa vencimientos, anomalías y hallazgos sin que el usuario pregunte |
| P4 | Todo relacionado | Grafo de conocimiento interno: trabajador ↔ equipo ↔ documento ↔ incidente |
| P5 | Auditable de fábrica | Cada acción firmada, versionada, trazable con WORM opcional |
| P6 | Regulatory-ready | Reportes exigidos por autoridad en 1 clic con estructura pre-validada |
| P7 | Diseño-primero | Estética Linear/Notion/Stripe. Nunca ERP feo |
| P8 | Sub-segundo | Búsqueda universal <500 ms P95, dashboard <1 s |

## 🏗 Arquitectura

12 bounded contexts (DDD): Identity, Workforce, Dosimetry, Equipment, Document Intelligence, Regulatory Compliance, Shielding, Nuclear Medicine, Instrumentation, Incidents, Audits, AI Copilot.

Ver `docs/architecture/RPMS_Arquitectura_Completa.md` para el blueprint completo (17 capítulos).

**Stack productivo:**
- **Backend:** .NET 9 (dominio) + Python 3.12 FastAPI (IA/OCR/RAG), arquitectura hexagonal + CQRS
- **Frontend:** Next.js 15 App Router + TypeScript strict + Tailwind + shadcn/ui
- **Datos:** PostgreSQL 16 (pgvector + Timescale + pgcrypto + PostGIS), Redis 7, OpenSearch 2, S3 con Object Lock (WORM)
- **Mensajería:** Kafka para eventos + Temporal para workflows regulatorios
- **IA:** Claude Sonnet/Opus (Copilot), bge-m3 / text-embedding-3-large (embeddings), LlamaIndex + LangGraph (agentes)
- **Infra:** Kubernetes (EKS/AKS/GKE), Terraform, ArgoCD, Istio
- **Observabilidad:** OpenTelemetry → Prometheus + Grafana + Loki + Tempo

## 📦 Estructura del monorepo

```
rpms/
├── docs/
│   ├── architecture/     # Blueprint (17 capítulos)
│   ├── adr/              # Architecture Decision Records
│   ├── api/              # Contratos OpenAPI 3.1 (pendiente)
│   ├── regulatory/       # Normativa CL/AR/PE/CO/MX + IAEA/ICRP
│   └── runbooks/         # Operación y respuesta a incidentes
├── libs/
│   └── rpms-core/        # RUT, UUID, Clock, EventBus, errores RFC 7807
├── services/
│   ├── workforce-svc/    # Trabajadores POE — HEXAGONAL COMPLETO
│   ├── dosimetry-svc/    # ICRP, anomalías, parsers dosimétricos
│   ├── document-svc/     # OCR + NER + reconciliación + RAG
│   ├── compliance-svc/   # Motor de reglas multi-jurisdicción
│   ├── identity-svc/     # (placeholder) OIDC + RBAC/ABAC/ReBAC
│   ├── notification-svc/ # (placeholder) email/SMS/WhatsApp/push
│   └── ai-orchestrator-svc/  # (placeholder) LangGraph + agentes
├── frontend/
│   ├── web/              # Next.js 15 con dashboard, ficha trabajador, ⌘K
│   └── mobile/           # (placeholder) React Native para inspecciones
├── db/
│   ├── migrations/postgres/  # DDL canónico (4 migraciones, 24 tablas)
│   ├── migrations/sqlite/    # Traducción automática para dev/test
│   └── scripts/          # apply_migrations.py + seed.py
├── infra/
│   ├── docker/           # Dockerfiles + docker-compose (13 servicios)
│   ├── k8s/              # Manifiestos con NetworkPolicies y HPA
│   ├── terraform/        # AWS: EKS + RDS + S3 (WORM) + Redis + KMS
│   └── observability/    # Prometheus scrape configs
├── scripts/              # Test runner unificado
├── .github/workflows/    # CI + security-nightly con SBOM y cosign
└── Makefile              # Targets: check-sandbox, db-init, seed
```

## 🚀 Quick start (sandbox sin red)

Requisitos mínimos: Python 3.12, sqlite3.

```bash
# 1. Aplicar migraciones a SQLite local
make db-init

# 2. Cargar seed de demo (Hospital San Rafael, 12 trabajadores, 72 lecturas)
make db-seed

# 3. Ejecutar suite de tests completa
make check-sandbox
```

Salida esperada:
```
======================================================================
RPMS — Suite de tests del monorepo
======================================================================
  [OK] libs/rpms-core/tests/test_core.py             (17 tests)
  [OK] services/workforce-svc/tests/test_domain.py   (26 tests)
  [OK] services/workforce-svc/tests/test_integration.py  (16 tests)
  [OK] services/document-svc/tests/test_pipeline.py  (29 tests)
  [OK] services/dosimetry-svc/tests/test_domain.py   (27 tests)
  [OK] services/compliance-svc/tests/test_rules.py   (14 tests)
======================================================================
Tests totales:       129
Fallos:              0
Errores:             0
```

## 🐳 Desarrollo full-stack (Docker Compose)

```bash
cd infra/docker
docker compose up -d              # 13 servicios
docker compose logs -f workforce-svc
```

Endpoints locales:
- **Web:** http://localhost:3000
- **Workforce API:** http://localhost:8001/docs
- **Document API:** http://localhost:8002/docs
- **Dosimetry API:** http://localhost:8003/docs
- **Compliance API:** http://localhost:8004/docs
- **Keycloak:** http://localhost:8080 (admin/admin)
- **MinIO Console:** http://localhost:9001
- **Grafana:** http://localhost:3001
- **Prometheus:** http://localhost:9090

## 🧪 Estado actual (v0.1.0)

**Implementado y probado (129 tests verdes):**

- ✅ Blueprint arquitectónico de 17 capítulos
- ✅ Base de datos PostgreSQL con 24 tablas + traductor a SQLite para dev
- ✅ Librería compartida rpms-core (UUID, RUT módulo 11, EventBus, RFC 7807)
- ✅ workforce-svc con arquitectura hexagonal completa
- ✅ dosimetry-svc: ICRP 103, detector de anomalías heurístico + IsolationForest
- ✅ compliance-svc: motor de reglas con 5 reglas chilenas (DS 133, DS 3, ISP)
- ✅ document-svc: pipeline OCR → clasificación → NER → reconciliación → RAG
- ✅ Frontend Next.js 15 con dashboard ejecutivo, ficha trabajador, command bar
- ✅ Docker Compose (13 servicios), Kubernetes manifests, Terraform AWS
- ✅ CI/CD GitHub Actions (SBOM + cosign) + security nightly

**Pendiente para el MVP comercial:**

- ⏳ identity-svc con Keycloak/OIDC + RBAC/ABAC/ReBAC
- ⏳ notification-svc: email + SMS + WhatsApp + push
- ⏳ ai-orchestrator-svc: LangGraph + Claude API + 6 agentes especializados
- ⏳ Ingesta dosimétrica automática (IMAP/SFTP) para 4 proveedores
- ⏳ Reportes regulatorios (5 formularios chilenos)
- ⏳ Módulos Blindajes / Medicina Nuclear / Instrumentación / Incidentes / Auditorías
- ⏳ Mobile app (Expo) para inspecciones offline con QR + firma biométrica
- ⏳ Portal del Trabajador y Portal Regulador
- ⏳ Integraciones HL7 FHIR R5 + DICOM RDSR + SCIM

## 🗺 Roadmap

| Fase | Duración | Objetivo |
|------|----------|----------|
| **0 — Fundaciones** | 0–1 mes | ✅ Setup, IaC, CI/CD, design system |
| **1 — MVP** | 2–4 meses | 🚧 Un hospital piloto reemplaza sus Excel |
| **2 — Comercial** | 5–8 meses | Ingesta automática, Copilot v1, mobile v1 |
| **3 — Enterprise** | 9–14 meses | Multi-jurisdicción, portal regulador, IA visual blindajes |
| **4 — Regional** | 15–20 meses | Certificación FDA/CE, partners, academia |

## 🔒 Cumplimiento normativo

**Chile** (implementado):
- DS N° 133/1984 — Autorizaciones para instalaciones y equipos radiactivos
- DS N° 3/1985 — Reglamento de protección radiológica
- DS N° 594/1999 — Condiciones sanitarias en lugares de trabajo
- Código Sanitario (Libro Sexto)
- Ley N° 18.302 — Seguridad Nuclear (CCHEN)
- Resoluciones específicas del ISP

**Internacional** (referencia):
- ICRP 60, 103, 118 — Límites de dosis, cristalino
- IAEA GSR Part 3 — Radiation Protection and Safety
- IAEA SSG-46 — Uso médico de radiaciones ionizantes
- NCRP 147/151 — Blindajes
- INES — International Nuclear Event Scale

**Multi-jurisdicción prevista** (fase 3): AR, PE, CO, MX, ES.

## 🔐 Seguridad

- Multi-tenant con Row-Level Security en todas las tablas
- Cifrado at-rest (KMS) + in-transit (mTLS interno via Istio)
- Object Lock GOVERNANCE en S3 para documentos regulatorios (10 años)
- WAF + rate limiting en edge
- Auditoría append-only (WORM) de cada acción
- MFA obligatorio para roles con escritura
- Certificaciones objetivo: **SOC 2 Type II + ISO 27001 + HIPAA**

## 📄 Licencia

Software propietario. Copyright © 2026 — Todos los derechos reservados. Uso mediante contrato comercial.

## 🔗 Referencias

- **Blueprint arquitectónico completo:** `docs/architecture/RPMS_Arquitectura_Completa.md`
- **ADRs:** `docs/adr/`
- **Documentación del frontend:** `frontend/web/README.md`
