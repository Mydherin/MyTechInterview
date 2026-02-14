# Clarifications Before Development

These are open questions and decisions that are useful to agree on before or early in development. Resolve them in requirements, ADRs, or team discussion as needed.

---

## 1. Multi-tenancy and authentication

- **Is there a single enterprise per deployment or multiple?**  
  Requirements say “each enterprise will have an instance of this application” – confirm whether that means one deployment per enterprise (no multi-tenancy) or one app that serves many enterprises with tenant isolation (e.g. by `enterpriseId`).

- **Who uses the back office?**  
  Only one “manager” per enterprise, or multiple roles (e.g. manager, receptionist, super-admin)? Do we need roles/permissions in v1?

- **Authentication:**  
  How do users log in? (e.g. email/password, SSO, magic link.) Any need for “invite user to enterprise” or “switch enterprise” in the same session?

---

## 2. Core entity details

- **Customer–gym relationship:**  
  Customers “can join any of the enterprise’s gyms.” Is the only link “through contracts” (each contract has `signedAtGymId`), or do we need an explicit “member of gym(s)” or “home gym” for access/visits?

- **Contract lifecycle:**  
  Can a customer have multiple **active** contracts at once (e.g. two rates, or two gyms), or must we enforce “at most one active contract per customer”? What exactly does “finish” mean (auto set `endDate`, block new usage, etc.)?

- **Employee–gym assignment:**  
  One gym per employee or many? Can an employee sign contracts at a gym they are not assigned to? (Current model: one `gymId` per employee; recruitment is any contract they signed.)

- **Rate structure:**  
  Only name and price, or do we need duration (e.g. monthly/yearly), type (e.g. “student”), or max gyms? Any business rules like “rate X only at gym Y”?

---

## 3. Filters and listing

- **Gym filters:**  
  “Name, location, and others” – which others? (e.g. city, status, number of contracts.)

- **Employee filters:**  
  Same idea: which “core filters” besides name? (e.g. gym, role, recruitment count.)

- **Rate filters:**  
  Which “essential filters”? (e.g. name, price range, active only.)

- **Pagination/sorting:**  
  Default page size, max page size, and default sort for each list (e.g. gyms by name, employees by name).

---

## 4. Real-time and events

- **“Real-time notification when a new customer has signed a contract”:**  
  Who receives it? (e.g. all back-office users of that enterprise, or only users of the gym where it was signed.)  
  In-app only, or also email/push?  
  “Event bus” – confirm if you mean a dedicated message broker (e.g. Redis Pub/Sub, RabbitMQ) or in-process events plus WebSockets/SSE for the SPA.

---

## 5. Analytics and ranking

- **Employee ranking “per month”:**  
  By contract **signed** date or by contract **start** date? Same rule for “average new customers per month”?

- **“Top 3 gyms with more customers”:**  
  Count distinct customers with at least one contract linked to that gym (by `signedAtGymId`), or only active contracts? Same for “most popular rates” (by contract count, active only or all time?).

- **Time range for analytics:**  
  All-time totals vs. configurable range (e.g. last 12 months)? Any need for comparison (e.g. this month vs last month)?

---

## 6. Non-functional and tech

- **API style:**  
  REST, or also GraphQL? Any preference for API versioning (e.g. `/v1/...`)?

- **Pipeline:**  
  “Backend pipeline” – confirm repo layout: monorepo (frontend + backend) or separate repos; and that “E2E” means backend API E2E (e.g. HTTP tests), not browser E2E.

- **Soft delete:**  
  Should soft-deleted entities appear in filters/lists at all (e.g. “show deleted” toggle for admins), or never?

---

## 7. UI/UX scope

- **Customer creation:**  
  Is a customer created only when the first contract is signed (contract form creates customer + contract), or can we create “customer” first and then add contracts? Same for “list customers” – do we need a dedicated customer list screen with contract history?

- **Notifications UX:**  
  Toast, bell icon with list, or both? Should notifications be persisted (e.g. “last 10” in DB) or only in-memory for the session?

---

Resolving these will reduce rework and keep the domain model and APIs aligned with real use cases. You can copy this file into your repo and tick items off as you decide.
