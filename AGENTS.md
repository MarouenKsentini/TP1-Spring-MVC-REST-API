# AGENTS.md — productapp

Spring Boot 4.1.2 + Java 17 + Maven wrapper. Single module, H2 in-memory, JPA / WebMVC / Thymeleaf. Root package is `com.example.productapp` (not `org.iset`).

## Commands

Run from the repo root with `.\mvnw.cmd` on Windows:

```bash
.\mvnw.cmd spring-boot:run                          # dev server
.\mvnw.cmd test                                     # all tests
.\mvnw.cmd -Dtest=ProductappApplicationTests test   # single test class
.\mvnw.cmd "-Dtest=ProductappApplicationTests#contextLoads" test  # single method
.\mvnw.cmd clean                                    # clear stale output
```

- The wrapper resolves Maven 3.10.0 and works; no local Maven install is needed.
- No `server.port` is configured, so the app listens on **8080**. A previous `spring-boot:run` may still hold that port (verify with `Get-NetTCPConnection -LocalPort 8080`). Kill the stale `com.example.productapp.ProductappApplication` process, or override: `.\mvnw.cmd spring-boot:run -Dspring-boot.run.arguments=--server.port=8081`.

## Wiring

- Entrypoint: `src/main/java/com/example/productapp/ProductappApplication.java`
- `controller/ProduitRestController.java` — JSON CRUD at `/api/produits`
- `controller/ProduitController.java` — MVC at `/produits`: `GET` renders `produits.html`, `POST /produits/add` saves and redirects back
- `service/ProduitService.java` — thin delegate over `ProduitRepository`
- `model/Produit.java` — entity: `Long id` (generated), `String libelle`, `Double prix`, `Integer qteStock`

The Thymeleaf form is wired to `ProduitController`; do not treat `/produits/add` as dead.

## REST API

All under `/api/produits`:

| Method | Path | Behavior |
|--------|------|----------|
| GET | `/api/produits` | 200, all products |
| GET | `/api/produits/{id}` | 200 or 404 |
| POST | `/api/produits` | 201 + created body |
| PUT | `/api/produits/{id}` | merges `libelle/prix/qteStock`; 200 or 404 |
| DELETE | `/api/produits/{id}` | 204 or 404 |

## H2

- `jdbc:h2:mem:testdb`, user `sa`, empty password.
- `spring.jpa.hibernate.ddl-auto=update` creates the schema on boot. There is no `data.sql` and no seed data.
- In-memory only: all data is lost on restart.
- Console at `/h2-console` (unsecured; do not expose publicly).

## Conventions

- Field injection via `@Autowired` (not constructor injection).
- `ProduitService.findById` returns `null` for missing rows; controllers check `== null` and map to 404.
- `PUT` copies only the three mutable fields; `id` is never overwritten.

## Tests

Only `ProductappApplicationTests.contextLoads()` exists — it boots the context and asserts nothing else. For real coverage prefer slice tests (`@WebMvcTest`, `@DataJpaTest`) over full `@SpringBootTest`.

## Gotchas

- Spring Boot 4 uses renamed starters: `spring-boot-starter-webmvc` (not `-web`), `spring-boot-h2console`, and test starters such as `spring-boot-starter-webmvc-test`. Match these when adding dependencies.
- No CI, lint, formatter, or code-quality plugins. `.github/modernize/java-upgrade` is Java-upgrade hook tooling, not CI.
