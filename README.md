# 🗳️ Election Management System (EMS) - Spring Boot Backend

![Java](https://img.shields.io/badge/Java-17-orange?style=for-the-badge&logo=java)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.1.5-green?style=for-the-badge&logo=spring)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)
![JWT](https://img.shields.io/badge/JWT-Authentication-red?style=for-the-badge&logo=json-web-tokens)
![Swagger](https://img.shields.io/badge/Swagger-OpenAPI-85EA2D?style=for-the-badge&logo=swagger)

A robust, scalable, and secure backend for an **Election Management System** built with Spring Boot. This system facilitates the entire election lifecycle, from voter registration and party creation to nomination processing, voting, and result computation.

---

## 📖 Table of Contents
- [System Architecture](#-system-architecture)
- [Entity Relationship Diagram](#-entity-relationship-diagram)
- [Features](#-features)
- [API Documentation (Interactive)](#-api-documentation)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Author](#-author)

---

## 🏗️ System Architecture

The application follows a standard **Layered Architecture** using Spring Boot. It utilizes Spring Security with JWT for stateless authentication and JPA/Hibernate for database interactions.

```mermaid
graph TD
    Client[📱 Client / Frontend] -->|HTTP Requests + Bearer Token| JwtFilter[🔒 JWT Authentication Filter]
    
    JwtFilter -->|Validates Token| Controllers
    
    subgraph Controllers [🌐 API Layer]
        AuthController[Auth Controller]
        ROController[Returning Officer Controller]
        PartyController[Party Controller]
        CandidateController[Candidate Controller]
        VoterController[Voter Controller]
    end
    
    Controllers -->|Business Logic| Services
    
    subgraph Services [⚙️ Service Layer]
        AuthService[Auth Service]
        ElectionService[Election Service]
        VotingService[Voting Service]
        ResultService[Result Service]
        NominationService[Nomination Service]
        PartyService[Party Service]
        VoterService[Voter Service]
        EmailService[Email Service]
    end
    
    Services -->|Data Access| Repositories
    
    subgraph Repositories [💾 Data Access Layer]
        UserRepo[User Repository]
        ElectionRepo[Election Repository]
        VoteRepo[Vote Repository]
        CandidateRepo[Candidate Repository]
        PartyRepo[Party Repository]
    end
    
    Repositories -->|JPA / Hibernate| DB[(🗄️ MySQL Database)]
    
    classDef filter fill:#f9d0c4,stroke:#333,stroke-width:2px;
    classDef controller fill:#d4e157,stroke:#333,stroke-width:2px;
    classDef service fill:#81d4fa,stroke:#333,stroke-width:2px;
    classDef repo fill:#ffcc80,stroke:#333,stroke-width:2px;
    classDef db fill:#b39ddb,stroke:#333,stroke-width:2px;
    
    class JwtFilter filter;
    class AuthController,ROController,PartyController,CandidateController,VoterController controller;
    class AuthService,ElectionService,VotingService,ResultService,NominationService,PartyService,VoterService,EmailService service;
    class UserRepo,ElectionRepo,VoteRepo,CandidateRepo,PartyRepo repo;
    class DB db;

🗄️ Entity Relationship Diagram

erDiagram
    USER ||--o| VOTER : "has profile"
    USER ||--o| CANDIDATE : "has profile"
    USER ||--o| PARTY : "leads"
    
    PARTY ||--o{ CANDIDATE : "nominates"
    
    CONSTITUENCY ||--o{ VOTER : "belongs to"
    CONSTITUENCY ||--o{ CANDIDATE : "contests in"
    CONSTITUENCY ||--o{ ELECTION : "hosts"
    
    ELECTION ||--o{ NOMINATION : "receives"
    ELECTION ||--o{ VOTE : "contains"
    ELECTION ||--o{ ELECTION_RESULT : "generates"
    
    CANDIDATE ||--o{ NOMINATION : "submits"
    CANDIDATE ||--o{ VOTE : "receives"
    CANDIDATE ||--o{ ELECTION_RESULT : "achieves"
    
    VOTER ||--o{ VOTE : "casts"
    
    USER {
        Long id
        String username
        String password
        String email
        Role role
        boolean verified
    }
    
    ELECTION {
        Long id
        String name
        LocalDateTime nominationStartDate
        LocalDateTime electionStartDate
        ElectionStatus status
    }
    
    NOMINATION {
        Long id
        String manifestoUrl
        NominationStatus status
    }
    
    VOTE {
        Long id
        LocalDateTime voteTime
        String transactionHash
    }

✨ Features
This system is divided into distinct roles, each with specific privileges.

<details> <summary><b>👤 Voter (Click to expand)</b></summary>
Self-registration with email verification.

Secure login using JWT.

View active elections based on their constituency.

Cast a secure vote (prevents double voting).

View live and final election results.

Update personal profile.

</details><details> <summary><b>🏛️ Party (Click to expand)</b></summary>
Register a new political party.

Add and manage party candidates.

View party-specific results and winning candidates.

Update party details.

</details><details> <summary><b>👨‍💼 Candidate (Click to expand)</b></summary>
Submit nomination forms (manifesto, affidavit links).

View own nomination status (Pending, Accepted, Rejected).

View election results.

</details><details> <summary><b>👨‍⚖️ Returning Officer (RO) (Click to expand)</b></summary>
Constituency Management: Create, update, delete, and search constituencies.

Election Management: Schedule elections, start/end nominations, start/end voting.

Voter Management: View statistics, verify voter registrations.

Nomination Management: Accept or reject candidate nominations.

Results: Trigger computation of election results and announce winners.

Party Management: Approve newly registered political parties.

</details>
🛠️ Interactive API Documentation
Note: All endpoints (except auth/register) require a Bearer <JWT_TOKEN> in the Authorization header.

<details> <summary><b>🔐 Authentication (`/api/auth`)</b></summary>
Method	Endpoint	Description
POST	/login	Authenticate user & generate JWT
GET	/verify?token=...	Verify email address
GET	/test-email-config	Test SMTP configuration
</details><details> <summary><b>👨‍⚖️ Returning Officer (`/api/ro`)</b></summary>
Constituencies:

POST /constituencies - Create constituency

GET /constituencies - Get all

PUT /constituencies/{id}/total-voters - Update voter count

Elections:

POST /elections - Schedule new election

PUT /elections/{id}/start-nomination - Start nomination phase

PUT /elections/{id}/end-election - End election

POST /elections/{id}/compute-results - Compute final results

Nominations & Voters:

GET /nominations/pending - View pending nominations

PUT /nominations/{id}/accept - Accept nomination

PUT /voters/{id}/verify - Verify a voter

</details><details> <summary><b>🏛️ Party (`/api/party`)</b></summary>
POST /register - Register a new party

POST /candidates - Add a candidate to the party

GET /candidates - Get party's candidates

GET /results - View results

</details><details> <summary><b>👨‍💼 Candidate (`/api/candidate`)</b></summary>
POST /nominate - Submit nomination

GET /nominations - Get own nominations

GET /results - View election results

</details><details> <summary><b>👤 Voter (`/api/voter`)</b></summary>
POST /register - Voter self-registration

POST /cast-vote - Cast a vote

GET /has-voted - Check if voted in a specific election

GET /profile - View profile

</details>
🚀 Getting Started
Prerequisites
Java 17 or higher

Maven

MySQL Server

Installation
Clone the repository

bash
git clone https://github.com/your-username/election-management-system.git
cd election-management-system
Database Setup
Create a MySQL database named election_db:

sql
CREATE DATABASE election_db;
Configure application.properties
Navigate to src/main/resources/application.properties and update the following:

properties
spring.datasource.url=jdbc:mysql://localhost:3306/election_db?useSSL=false&serverTimezone=Asia/Kolkata&allowPublicKeyRetrieval=true
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD

# Email Configuration (for verification links)
spring.mail.username=YOUR_EMAIL@gmail.com
spring.mail.password=YOUR_APP_PASSWORD
Build and Run

bash
mvn clean install
mvn spring-boot:run
The server will start on http://localhost:9000.

Access Swagger UI
Open your browser and navigate to:
http://localhost:9000/swagger-ui/index.html

🔒 Environment Variables (Best Practices)
For production deployment, never commit your passwords. Use environment variables:

Variable	Description
SPRING_DATASOURCE_USERNAME	MySQL database username
SPRING_DATASOURCE_PASSWORD	MySQL database password
SPRING_MAIL_USERNAME	Gmail/SMTP username
SPRING_MAIL_PASSWORD	Gmail App password
JWT_SECRET	Base64 encoded secret key for JWT
👨‍💻 Author
Sabarish M S

Salem, Tamil Nadu, India

GitHub: @SabarishMS123

Portfolio: sabarishdev.netlify.app

© All rights reserved.
