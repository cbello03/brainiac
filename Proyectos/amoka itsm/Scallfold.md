```bash
quarkus create app com.starveil.nexus:amoka-backend --extension='resteasy-reactive,resteasy-reactive-jackson,hibernate-orm-panache,jdbc-postgresql,agroal,smallrye-jwt,quarkus-liquibase,quarkus-security,quarkus-smallrye-openapi' --no-code
```




Perfect. Let’s create a **full Quarkus MVP backend project** for your ITSM app using **Maven, Java 21**, and all the extensions you’ll need for **JWT auth, PostgreSQL, users/roles/permissions, and REST APIs**.  

Here’s the **Quarkus CLI command**:  

```bash
quarkus create app com.montara.itsm \
    --extension='resteasy-reactive,resteasy-reactive-jackson,hibernate-orm-panache,jdbc-postgresql,agroal,smallrye-jwt,quarkus-liquibase,quarkus-security' \
    --build-tool=maven \
    --no-code
```

---

### **Explanation**
| Option | Meaning |
|--------|---------|
| `com.montara.itsm` | **Group ID + artifact ID** (project namespace and JAR name) |
| `--extension` | Adds **all needed Quarkus extensions**: REST API, JSON, PostgreSQL, ORM, JWT, database migration, security |
| `--build-tool=maven` | Use Maven for simplicity and enterprise stability |
| `--no-code` | Start with a clean scaffold without example endpoints; you’ll add your own |

---

### **After Generation**
1. Move into your project:
```bash
cd itsm
```
2. Run dev mode with hot reload:
```bash
./mvnw quarkus:dev
```
- Dev server: `http://localhost:8080/`  
- OpenAPI docs (optional): add `quarkus-smallrye-openapi` extension and use `http://localhost:8080/q/swagger-ui`  

---

💡 **Optional Extensions for Later**
- `quarkus-cache` → caching for incidents/users  
- `quarkus-hibernate-validator` → validate payloads  
- `quarkus-mailer` → for notifications  

---

If you want, I can **create the full folder structure + entities for JWT auth, users, roles, permissions, and a skeleton Incident Management module**, ready to start coding immediately.  

Do you want me to do that next?