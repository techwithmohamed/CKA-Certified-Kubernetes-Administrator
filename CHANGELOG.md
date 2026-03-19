# Changelog

All notable changes to this CKA prep repo are documented here.

Format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [1.1.0] — 2026-03-19

### Added
- 5 new exercises: Helm install/upgrade (#13), Kustomize overlays (#14), Gateway API (#15), HPA (#16), kubectl debug (#17)
- 5 new YAML skeletons: hpa, limitrange, serviceaccount, sidecar-init-container, validatingadmissionpolicy
- 6 new troubleshooting scenarios: Helm release stuck, Gateway API misconfigured, native sidecar not starting, kubectl debug not working, CSI volume mount failure, Kustomize apply not working
- Local YAML validation script (`scripts/validate-local.sh`)
- Mock exam self-scoring worksheet in README
- CHANGELOG.md

### Changed
- Exercises index updated to cover 17 exercises across all 5 CKA domains
- Skeletons index updated to list 23 templates
- README repo structure tree updated
- README badges expanded (Kubernetes version, exercise count)

### Fixed
- Repository URLs migrated from old GitHub org to theplatformlab

---

## [1.0.0] — 2026-03-01

### Added
- 12 hands-on exercises covering core CKA domains
- 18 YAML skeletons for exam-day quick recall
- kubectl cheat sheet
- Troubleshooting playbook (9 common scenarios)
- 17-question mock exam weighted to real CKA distribution
- Exam setup script with aliases, vim config, bash completion
- CI pipeline for YAML linting
- Community templates: issue templates, PR template, CONTRIBUTING.md
- Study progress tracker
- 30/45/60-day study plans
