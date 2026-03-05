# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

学之思在线考试系统 (XZS Online Exam System) — PostgreSQL edition. A Java + Vue.js full-stack exam platform with:
- **Backend**: Spring Boot 2.1.6 (Java 8) on port 8000
- **Student Frontend**: Vue 2 + Element UI on port 8001
- **Admin Frontend**: Vue 2 + Element UI on port 8002

### System Dependencies (pre-installed)

- Java 8 (`openjdk-8-jdk`) — set `JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`
- Maven 3.8+
- PostgreSQL 16 (database: `xzs`, user: `postgres`, password: `123456`)
- Redis (default port 6379)
- Node.js 14 via nvm (required for `node-sass@4.14.1` compatibility)

### Starting Services

1. **PostgreSQL**: `sudo pg_ctlcluster 16 main start`
2. **Redis**: `sudo redis-server --daemonize yes`
3. **Backend**: From `source/xzs/`, build with `mvn clean package -DskipTests`, then run:
   ```
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
   java -Duser.timezone=Asia/Shanghai \
     -Dspring.redis.host=localhost \
     -Dspring.datasource.url=jdbc:postgresql://localhost:5432/xzs \
     -Dspring.datasource.username=postgres \
     -Dspring.datasource.password=123456 \
     -jar target/xzs-3.2.0.jar
   ```
4. **Frontend dev servers**: Use `nvm use 14` before running. From each Vue app dir:
   ```
   npm run serve
   ```
   - Student: `source/vue/xzs-student` (port 8001)
   - Admin: `source/vue/xzs-admin` (port 8002)

### Important Gotchas

- **`application-dev.yml` points to `192.168.0.96`** — must override with JVM args (`-Dspring.redis.host=localhost`, `-Dspring.datasource.url=...`) when running locally. Do NOT modify the file.
- **Passwords use RSA encryption**, not BCrypt. The stored password in DB is RSA-encrypted using the public key in `application.yml`. During login, it is decrypted with the private key and compared to the plaintext from the request.
- **Redis caching** — if you change user data in PostgreSQL directly (e.g., updating passwords), flush Redis with `redis-cli FLUSHALL` to avoid stale cached user objects.
- **No SQL init script in repo** — the database schema must be created manually based on domain entities and mapper XMLs. A schema was initialized during setup.
- **Node.js 14 is required** for the Vue frontends due to `node-sass@4.14.1`. Use `nvm use 14` before `npm install` or `npm run serve`.
- **Student frontend lint** has 1 pre-existing error in `src/api/question.js` (unused import). Admin frontend lint passes clean.
- **Login credentials**: student/123456, admin/123456.
- Both Vue frontends proxy `/api/*` to `http://localhost:8000` via webpack-dev-server config in `vue.config.js`.

### Lint / Build / Test Commands

- **Backend build**: `cd source/xzs && mvn clean package -DskipTests`
- **Backend compile check**: `cd source/xzs && mvn clean compile`
- **Frontend lint**: `cd source/vue/xzs-student && npm run lint` or `cd source/vue/xzs-admin && npm run lint`
- **Frontend build**: `cd source/vue/xzs-student && npm run build` or `cd source/vue/xzs-admin && npm run build`
- Maven tests are skipped by default in `pom.xml` (`maven-surefire-plugin` with `skipTests=true`).
