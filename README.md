# 🚀 Docker로 JAR 최적화 & 컨테이너 실행

### 📀 Docker 환경에서 JAR 파일 실행 실습
   Spring Boot 애플리케이션을 JAR 파일로 패키징한 후, Docker 환경에서 최적화된 이미지로 빌드하고 컨테이너로 실행하는 방법을 학습

---
   
   #### ✅ 주요 실습 내용
   <br>
   
   **📌JAR 파일 생성**
   * Spring Tool Suite(STS)에서 JAR 파일 빌드
   * Run Configuration을 활용한 실행 및 폴더 지정
   * 프로젝트의 Properties에서 JAR 파일 경로 확인
   
   <br>
   
   **📌Docker를 활용한 JAR 실행**
   * Dockerfile 최적화 기법
   * Docker 이미지 생성 및 컨테이너 실행
   * Docker Hub에 이미지 업로드 및 다운로드 후 실행

<br>

### 1. STS(Spring Tool Suite)에서 JAR 파일 만드는 방법

---

1. **Run Configurations 실행**  
   - STS에서 `Run` → `Run Configurations` 또는 `Debug Configurations` 실행  

2. **폴더 이름 지정**  
   - `Run Configurations` 창에서 `Classpath` 탭 선택  
   - `User Entries` 아래에 폴더(출력 경로) 지정
    <img src="https://github.com/user-attachments/assets/3e10d765-a935-4b69-9931-dfca210781c5" width="500">

3. **프로젝트의 Properties에서 경로 확인**  
   - 프로젝트를 우클릭 → `Properties` → `Java Build Path` → `Libraries` 또는 `Classpath` 확인  

4. **JAR 파일 및 라이브러리(lib) 폴더 확인**  
   - 지정한 폴더에 `lib` 폴더 생성됨  
   - `.jar` 파일이 해당 폴더 내에 존재하는지 확인
    <img src="https://github.com/user-attachments/assets/3556b5d5-2b8f-44ff-b7ba-bb7c21835bff" width="500">

5. **JAR 파일 실행 가능 여부 테스트**  
   - `java -jar 파일명.jar` 명령어로 실행 가능 여부 확인  

---

<br>

### 2. dockerfile의 최적화 방식
---
**1. `openjdk:17-slim`**

✔️ **특징**  
- `openjdk:17`을 기반으로 한 **경량화된 이미지**  
- `Debian`을 최소화하여 불필요한 패키지를 줄임  
- `glibc`를 포함하여 널리 사용되는 Linux 라이브러리를 지원  

✔️ **장점**  
- **일반적인 Java 애플리케이션** 실행에 적합  
- 안정적인 라이브러리 제공  

❌ **단점**  
- `Alpine` 기반 이미지보다 용량이 큼  
- 보안 패치가 `Alpine`보다 상대적으로 느릴 수 있음  

---

**2. `openjdk:17-alpine`**

✔️ **특징**  
- `Alpine Linux` 기반의 초경량 JDK 이미지  
- `musl libc` 사용 (glibc 미포함)  
- **최소한의 라이브러리**만 포함하여 보안성과 경량성을 높임  

✔️ **장점**  
- **가장 가벼운 JDK 이미지** (10MB~20MB 수준)  
- 보안 패치가 빠르고, **컨테이너 보안성 강화**  

❌ **단점**  
- 일부 Java 라이브러리가 `musl` 환경에서 호환되지 않을 수 있음  
- 특정 네이티브 라이브러리를 사용하는 애플리케이션 실행 시 문제 발생 가능  

---

**3. `eclipse-temurin:17-jdk-alpine AS builder`**

✔️ **특징**  
- `Eclipse Temurin`(Adoptium) JDK를 **Alpine 기반**으로 제공  
- `AS builder`: **멀티 스테이지 빌드**에서 사용되는 빌더 이미지  
- 빌드 도구(예: Maven, Gradle)를 포함할 수 있음  

✔️ **장점**  
- `openjdk`보다 **최적화된 성능과 유지보수 지원**  
- 빌드에 필요한 툴을 포함할 수 있어 **멀티 스테이지 빌드에 적합**  

❌ **단점**  
- 실행 환경이 아닌 빌드 환경이므로 최종 컨테이너에는 적합하지 않음  
- 실행 파일 크기가 커질 수 있음  

---

**4. `eclipse-temurin:17-jre`**

✔️ **특징**  
- `Eclipse Temurin`(Adoptium)에서 제공하는 **JRE(Java Runtime Environment) 전용 이미지**  
- **JDK가 필요 없는 애플리케이션 실행에 적합**  
- 최소한의 Java 실행 환경 제공  

✔️ **장점**  
- **불필요한 JDK 파일 제거로 이미지 크기 절감**  
- 실행만 필요한 환경에서는 JRE가 더 적합함  

❌ **단점**  
- **Java 빌드를 수행할 수 없음** (JDK 없음)  
- 일부 애플리케이션은 JDK를 필요로 할 수 있음  

---

**📌 최적의 선택은?**

| 이미지 | 크기 | 특징 | 권장 사용 |
|--------|------|------|------------|
| **`openjdk:17-slim`** | 중간 (~50MB) | `glibc` 포함, 안정적 | 일반적인 애플리케이션 실행 |
| **`openjdk:17-alpine`** | 가벼움 (~20MB) | `musl libc` 기반, 보안성 높음 | **경량 컨테이너**를 원할 때 |
| **`eclipse-temurin:17-jdk-alpine AS builder`** | 큼 (~200MB) | 빌드 환경 포함 | **멀티 스테이지 빌드**에서 사용 |
| **`eclipse-temurin:17-jre`** | 가장 작음 (~15MB) | 실행 환경만 포함 | **JRE 기반 실행만 필요할 때** |

✅ **운영 환경** 에서는 **`openjdk:17-alpine`** 또는 **`eclipse-temurin:17-jre`** 가 가장 적합  
✅ **빌드 환경** 에서는 **`eclipse-temurin:17-jdk-alpine AS builder`** 가 최적  

---

<br>

### 3. dockerfile 실제 작성 및 비교

   ---

<img src="https://github.com/user-attachments/assets/f288d6c2-c778-4cd5-bf37-5f9b7a211005" width="500">

<br>

**openjdk:17-slim**

<img src="https://github.com/user-attachments/assets/73f55581-9aac-48fa-9225-612712a3ec10" width="500">

<br>

**openjdk:17-alpine**

<img src="https://github.com/user-attachments/assets/4db61d50-4d12-4b90-b637-5bb4d960c2ae" width="500">

<br>

**eclipse-temurin:17-jdk-alpine AS builder**

<img src="https://github.com/user-attachments/assets/3046b7df-391b-4c9d-901c-d73b6e74c833" width="500">

<br>

**eclipse-temurin:17-jre**


   
   #### 최종 결과

   ![image](https://github.com/user-attachments/assets/fafe3395-d323-4ce2-9770-78299a094f08)

   
---

### 4. 이미지 생성법
  ---
  
   **Docker 이미지 생성법: Commit 방식 vs Build 방식**
   
   Docker 이미지를 생성하는 방법은 크게 **`commit 방식`** 과 **`build 방식(Dockerfile 사용)`**
   두가지가 있습니다.

   <br>

   #### **✅ 1. Commit 방식 (컨테이너 수정 후 이미지 생성)**

   🔹 컨테이너에서 직접 수정한 내용을 새로운 이미지로 저장하는 방식입니다.
   🔹 간단한 변경에는 유용하지만, 자동화가 어렵고 관리가 불편할 수 있습니다.

   #### 🔷 **사용 방법**

   ```bash
   # 기존 이미지로 컨테이너 실행 (인터랙티브 모드)
docker run -it --name myapp-container ubuntu:latest /bin/bash

# 컨테이너 내부에서 패키지 설치 또는 환경 설정 변경
apt update && apt install -y curl
exit  # 컨테이너 종료

# 변경된 컨테이너를 새로운 이미지로 저장
docker commit myapp-container myapp-custom:latest

# 새로 생성된 이미지 확인
docker images

# 생성한 이미지로 컨테이너 실행
docker run -d --name new-container -p 8080:8080 myapp-custom:latest

```

#### **✅ Commit 방식의 장점 & 단점**

✔️ **장점:**

- 빠르게 기존 이미지를 수정하여 새로운 이미지 생성 가능
- Dockerfile 없이 즉시 이미지 생성 가능

❌ **단점:**

- 변경 내역을 추적하기 어려움
- 자동화 및 재현성 부족
- 빌드 과정이 비효율적

---

#### **✅ 2. Build 방식 (Dockerfile 사용)**

🔹 Dockerfile을 이용하여 이미지를 **자동화된 방법**으로 생성하는 방식입니다.
🔹 변경 내역을 추적할 수 있어 **버전 관리가 용이**하며, CI/CD 환경에서 주로 사용됩니다.

#### 🔷 **사용 방법**

```
# Base 이미지 선택
FROM eclipse-temurin:17-slim-alpine

# 작업 디렉토리 설정
WORKDIR /app

# 애플리케이션 파일 복사
COPY myapp.jar app.jar

# 컨테이너 시작 시 실행할 명령어
CMD ["java", "-jar", "app.jar"]

```

```bash
# Docker 이미지 빌드
docker build -t myapp:latest .

# 빌드한 이미지로 컨테이너 실행
docker run -d --name myapp-container -p 8080:8080 myapp:latest

```

#### **✅ Build 방식의 장점 & 단점**

✔️ **장점:**

- 변경 내역을 명확하게 기록할 수 있음
- 자동화 가능 (CI/CD 연동)
- 최적화된 빌드 가능

❌ **단점:**

- Dockerfile을 작성해야 함
- 설정이 복잡할 수 있음

---

#### **🎯 Commit vs Build 방식 비교**

| 비교 항목 | **Commit 방식** | **Build 방식 (Dockerfile)** |
| --- | --- | --- |
| **사용 목적** | 빠른 수정 & 테스트 | 자동화, 최적화, 버전 관리 |
| **재현 가능성** | 낮음 | 높음 |
| **자동화** | 불가능 | 가능 (CI/CD 적용 가능) |
| **권장 사용 사례** | 단순한 변경 | 체계적인 이미지 관리 |

#### 결론 
   Commit 방식은 테스트용으로, Build 방식은 실제 운영 환경에 적합함으로 실무에 가까운 경험을 하기
   위해 Build 방식을 사용
   ![image](https://github.com/user-attachments/assets/ef97585b-0f7e-411f-a4e4-879a3e8a26c6)

---

<br>

### 5. 생성된 이미지로 컨테이너 만드는 법<br>

   ```bash
# 이미지 생성 (Dockerfile 기반)
docker build -t myapp:latest .

# 생성된 이미지로 컨테이너 실행
docker run -d --name myapp-container -p 8080:8080 myapp:latest

# 실행 중인 컨테이너 확인
docker ps

# 컨테이너 로그 확인
docker logs myapp-container

# 컨테이너 내부 접속
docker exec -it myapp-container /bin/bash

# 컨테이너 중지 및 삭제
docker stop myapp-container
docker rm myapp-container

# Docker Hub에서 이미지 다운로드 (Pull)
docker pull mydockerhubid/myapp:latest

```


   
   ![image](https://github.com/user-attachments/assets/8d53495c-74b4-4d0c-8c27-1dddf17f2f9c)

<br>

### 6. 이미지를 docker hub에 올리는 법<br>

``` bash 
# Docker Hub 로그인
docker login

# 이미지 태그 추가
docker tag myapp:latest mydockerhubid/myapp:latest

# 이미지 업로드 (Push)
docker push mydockerhubid/myapp:latest
```
![image](https://github.com/user-attachments/assets/bd354548-95e0-4950-add8-33b506226b26)

<br>

### 7. hub에서 이미지를 다운받고 실행하는 법<br>

```bash
# 다운로드한 이미지로 컨테이너 실행
docker run -d --name myapp-container -p 8080:8080 mydockerhubid/myapp:latest
```
   ![image](https://github.com/user-attachments/assets/43da8f86-391b-434f-83cf-c2e768828ad1) <br>
   ![image](https://github.com/user-attachments/assets/63c9f0ed-8ead-4d52-b38c-39dbf72dac3e)


