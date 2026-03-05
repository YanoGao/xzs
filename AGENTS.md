# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

学之思在线考试系统 (XZS Online Examination System) - PostgreSQL版. A Java + Vue.js exam platform with 3 sub-projects under `source/`:

| Service | Path | Port | Tech |
|---|---|---|---|
| Backend API | `source/xzs/` | 8000 | Java 8 / Spring Boot 2.1.6 / Maven |
| Admin Frontend | `source/vue/xzs-admin/` | 8002 | Vue 2.6 / Element UI / Vue CLI 4 |
| Student Frontend | `source/vue/xzs-student/` | 8001 | Vue 2.6 / Element UI / Vue CLI 4 |

### Prerequisites (installed via VM snapshot)

- **Java 8** (`JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`, set in `~/.bashrc`)
- **Maven 3.8.7**
- **Node.js 14** via nvm (`nvm use 14` or `source ~/.nvm/nvm.sh && nvm use 14`)
- **PostgreSQL 16** (database `xzs`, user `postgres`, password `123456`)
- **Redis** (default port 6379)

### Starting services

1. **PostgreSQL**: `sudo pg_ctlcluster 16 main start`
2. **Redis**: `sudo redis-server --daemonize yes`
3. **Backend**: `cd source/xzs && export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 && mvn clean package -DskipTests -q && java -Duser.timezone=Asia/Shanghai -jar -Dspring.profiles.active=dev target/xzs-3.2.0.jar`
4. **Admin Frontend**: `cd source/vue/xzs-admin && source ~/.nvm/nvm.sh && nvm use 14 && npm run serve`
5. **Student Frontend**: `cd source/vue/xzs-student && source ~/.nvm/nvm.sh && nvm use 14 && npm run serve`

### Key gotchas

- The backend requires **Java 8** specifically. Java 21 (system default before setup) is not compatible with Spring Boot 2.1.6. Always use `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64` before Maven or Java commands.
- The Vue frontends use `node-sass@4.14.1` which requires **Node.js 14**. Higher Node versions will fail during `npm install`. Always run `nvm use 14` before working with the frontends.
- `application-dev.yml` configures PostgreSQL and Redis to `localhost`. The original repo points to `192.168.0.96` — ensure it is set to `localhost` for local development.
- Passwords in the database are **RSA-encrypted** (not BCrypt). The RSA keys are in `application.yml` under `system.pwdKey`.
- The repo does not include SQL migration scripts. The database schema was created from MyBatis mapper definitions and `docs/database.md`.
- Default accounts: `admin/123456` (role=3, admin), `student/123456` (role=1, student).
- Both Vue frontends proxy `/api/*` to `http://localhost:8000` (the backend).
- The student frontend has a pre-existing lint error in `src/api/question.js` (unused import). This is not a regression.

### Lint / Test / Build

- **Admin lint**: `cd source/vue/xzs-admin && npx vue-cli-service lint`
- **Student lint**: `cd source/vue/xzs-student && npx vue-cli-service lint`
- **Backend build**: `cd source/xzs && mvn clean package -DskipTests`
- **Frontend build**: `cd source/vue/xzs-admin && npm run build` / `cd source/vue/xzs-student && npm run build`
- Backend tests are skipped by default in `pom.xml` (`maven-surefire-plugin` with `skipTests=true`).
