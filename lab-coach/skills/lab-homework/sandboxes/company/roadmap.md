---
type: sandbox-context
sandbox: company
updated: 2026-04-09
---

# Roadmap — Current Quarter

## Q2 Priorities (from Natasha)

### P0 — Must ship
1. **{{domain}} workflow redesign** — the core user flow is 4 years old and customers complain. Design is 70% done (Ira), implementation not started. Target: end of quarter.
2. **SOC 2 compliance** — legal requirement for enterprise customers. Mostly process + some technical controls. Platform team leads, but your team has 3 tasks.

### P1 — Should ship
3. **API v2** — current API is inconsistent and poorly documented. Kirill owns the design. You're reviewing. Target: beta by end of quarter.
4. **Performance improvements** — main dashboard loads in 4s, should be under 1s. No owner yet. Data team says it's a frontend problem, frontend says it's the API.

### P2 — If time allows
5. **Self-serve onboarding** — currently, new customers go through a sales-assisted setup. Product wants to automate this. Design not started.
6. **Internal tooling** — admin panel is held together with duct tape. Everyone wants it fixed, nobody wants to do it.

## Technical debt

- Test suite: 60% coverage, but many tests are flaky or test implementation details
- Database migrations: 3 pending migrations that nobody wants to run in production
- Monitoring: alerts exist but are noisy — team ignores most of them
- Documentation: API docs are auto-generated and wrong in places

## Dependencies

- {{domain}} workflow redesign depends on Ira finishing design and API v2 being stable
- SOC 2 depends on platform team, who are also doing a Kubernetes migration
- Performance improvements depend on data team's analytics pipeline changes

## The unspoken tension

The roadmap has 6 months of work and 3 months of quarter left. Something will get cut. The question is what — and who makes that call.
