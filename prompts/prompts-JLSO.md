# Prompts

| Author | Jorge Luis Sánchez Ocampo |
| --- | --- |
| AI Assistant | Chat-GPT && Github-Copilot(Claude 3.5 Sonnet) |
| Project | https://github.com/IamJorx/AI4Devs-db.git |

```markdown
Actua como un Architecto de Sistemas y DBA experto. Requiero que crees el código 
sql del siguiente ERD(entity relational diagram), que está en formato mermaid, 
para actualizar mi base de datos incluyendo las entidades que está en el ERD: 
	
	"
	erDiagram
     COMPANY {
         int id PK
         string name
     }
     EMPLOYEE {
         int id PK
         int company_id FK
         string name
         string email
         string role
         boolean is_active
     }
     POSITION {
         int id PK
         int company_id FK
         int interview_flow_id FK
         string title
         text description
         string status
         boolean is_visible
         string location
         text job_description
         text requirements
         text responsibilities
         numeric salary_min
         numeric salary_max
         string employment_type
         text benefits
         text company_description
         date application_deadline
         string contact_info
     }
     INTERVIEW_FLOW {
         int id PK
         string description
     }
     INTERVIEW_STEP {
         int id PK
         int interview_flow_id FK
         int interview_type_id FK
         string name
         int order_index
     }
     INTERVIEW_TYPE {
         int id PK
         string name
         text description
     }
     CANDIDATE {
         int id PK
         string firstName
         string lastName
         string email
         string phone
         string address
     }
     APPLICATION {
         int id PK
         int position_id FK
         int candidate_id FK
         date application_date
         string status
         text notes
     }
     INTERVIEW {
         int id PK
         int application_id FK
         int interview_step_id FK
         int employee_id FK
         date interview_date
         string result
         int score
         text notes
     }

     COMPANY ||--o{ EMPLOYEE : employs
     COMPANY ||--o{ POSITION : offers
     POSITION ||--|| INTERVIEW_FLOW : assigns
     INTERVIEW_FLOW ||--o{ INTERVIEW_STEP : contains
     INTERVIEW_STEP ||--|| INTERVIEW_TYPE : uses
     POSITION ||--o{ APPLICATION : receives
     CANDIDATE ||--o{ APPLICATION : submits
     APPLICATION ||--o{ INTERVIEW : has
     INTERVIEW ||--|| INTERVIEW_STEP : consists_of
     EMPLOYEE ||--o{ INTERVIEW : conducts
 
	"
```

```markdown
Puedes mejorar esta estructura y normalizarla
```

```markdown
Confirma si el código generado en compatible con postgresql
```

```markdown
Puedes indicar que indices seran necesarios para este sistema
```

```markdown
puedes unificar todos los scripts sql en un solo bloque, convierte los títulos en comentarios sql
```

```sql
Explícame la parte de prisma del siguiente texto, y que harías para cumplir esa tarea?: "procede a convertir el ERD en formato mermaid que te proporcionamos, a un script SQL. Analiza la base de datos del código actual y el script SQL y expande la estructura de datos usando las migraciones de Prisma."
```

```markdown
puedes incluir lo que sea nuevo del siguiente schema de prisma, dentro del archivo #file:schema.prisma, no alteres lo que ya está en este archivo, solo incluye lo nuevo: "generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL") // Configura la URL de conexión a tu base de datos
}

model Company {
  id        Int       @id @default(autoincrement())
  name      String
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  employees Employee[]
  positions Position[]
}

model Employee {
  id        Int      @id @default(autoincrement())
  companyId Int
  name      String
  email     String   @unique
  role      String
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  company Company @relation(fields: [companyId], references: [id])
  interviews Interview[]
}

model PositionStatus {
  id   Int    @id @default(autoincrement())
  name String @unique
  positions Position[]
}

model EmploymentType {
  id        Int       @id @default(autoincrement())
  name      String    @unique
  positions Position[]
}

model Position {
  id                 Int              @id @default(autoincrement())
  companyId          Int
  interviewFlowId    Int
  title              String
  description        String?
  statusId           Int
  isVisible          Boolean          @default(true)
  location           String?
  jobDescription     String?
  requirements       String?
  responsibilities   String?
  salaryMin          Float?
  salaryMax          Float?
  employmentTypeId   Int
  benefits           String?
  companyDescription String?
  applicationDeadline DateTime?
  contactInfo        String?
  createdAt          DateTime         @default(now())
  updatedAt          DateTime         @updatedAt

  company       Company       @relation(fields: [companyId], references: [id])
  interviewFlow InterviewFlow @relation(fields: [interviewFlowId], references: [id])
  status        PositionStatus @relation(fields: [statusId], references: [id])
  employmentType EmploymentType @relation(fields: [employmentTypeId], references: [id])
  applications  Application[]
}

model InterviewFlow {
  id           Int             @id @default(autoincrement())
  description  String
  createdAt    DateTime        @default(now())
  updatedAt    DateTime        @updatedAt
  positions    Position[]
  steps        InterviewStep[]
}

model InterviewType {
  id          Int             @id @default(autoincrement())
  name        String          @unique
  description String?
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  steps       InterviewStep[]
}

model InterviewStep {
  id              Int           @id @default(autoincrement())
  interviewFlowId Int
  interviewTypeId Int
  name            String
  orderIndex      Int
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt

  interviewFlow InterviewFlow @relation(fields: [interviewFlowId], references: [id])
  interviewType InterviewType @relation(fields: [interviewTypeId], references: [id])
  interviews    Interview[]
}

model Candidate {
  id        Int      @id @default(autoincrement())
  firstName String
  lastName  String
  email     String   @unique
  phone     String?
  address   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  applications Application[]
}

model ApplicationStatus {
  id          Int         @id @default(autoincrement())
  name        String      @unique
  applications Application[]
}

model Application {
  id           Int       @id @default(autoincrement())
  positionId   Int
  candidateId  Int
  applicationDate DateTime @default(now())
  statusId     Int
  notes        String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  position   Position   @relation(fields: [positionId], references: [id])
  candidate  Candidate  @relation(fields: [candidateId], references: [id])
  status     ApplicationStatus @relation(fields: [statusId], references: [id])
  interviews Interview[]
}

model InterviewResult {
  id         Int        @id @default(autoincrement())
  name       String     @unique
  interviews Interview[]
}

model Interview {
  id              Int           @id @default(autoincrement())
  applicationId   Int
  interviewStepId Int
  employeeId      Int
  interviewDate   DateTime
  resultId        Int?
  score           Int?          @default(0)
  notes           String?
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt

  application   Application   @relation(fields: [applicationId], references: [id])
  interviewStep InterviewStep @relation(fields: [interviewStepId], references: [id])
  employee      Employee      @relation(fields: [employeeId], references: [id])
  result        InterviewResult? @relation(fields: [resultId], references: [id])
}"
```

```markdown
ahora hagamos la migración de lo que incluimos en #file:schema.prisma hacia la base de datos. Dime los pasos que vas a implementar para realizar la migración:
```

```markdown
analiza el archivo #file:README.md e identifica que pasos son necesarios para garantizar la conexión con la base de datos, para poder hacer la migración con prisma
```

```markdown
dame el dato resultante de database_url al reemplazar todos los valores: DB_PASSWORD=D1ymf8wyQEGthFR1E9xhCq
DB_USER=LTIdbUser
DB_NAME=LTIdb
DB_PORT=5432
DATABASE_URL="postgresql://${DB_USER}:${DB_PASSWORD}@localhost:${DB_PORT}/${DB_NAME}"
```

```markdown
dime como puedo hacer para crear los indices que generamos en el código sql, necesito ejecutarlos directamente sobre una consola de la base de datos, o puedo hacerlo a través de prisma. Dime también si es que ya los incluiste en el schema de prisma
```

```markdown
actualiza el archivo #file:schema.prisma con el siguiente schema, actualiza solo lo nuevo, no actualices el dato url del objeto db. Aquí el nuevo schema de prisma con algunas actualizaciones: "generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Company {
  id        Int       @id @default(autoincrement())
  name      String
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  employees Employee[]
  positions Position[]

  @@index([name]) // Índice para búsquedas rápidas por nombre de la empresa
}

model Employee {
  id        Int      @id @default(autoincrement())
  companyId Int
  name      String
  email     String   @unique
  role      String
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  company Company @relation(fields: [companyId], references: [id])
  interviews Interview[]

  @@index([companyId]) // Índice para búsquedas rápidas por empresa
  @@index([role])      // Índice para búsquedas por rol
}

model PositionStatus {
  id   Int    @id @default(autoincrement())
  name String @unique
  positions Position[]
}

model EmploymentType {
  id        Int       @id @default(autoincrement())
  name      String    @unique
  positions Position[]
}

model Position {
  id                 Int              @id @default(autoincrement())
  companyId          Int
  interviewFlowId    Int
  title              String
  description        String?
  statusId           Int
  isVisible          Boolean          @default(true)
  location           String?
  jobDescription     String?
  requirements       String?
  responsibilities   String?
  salaryMin          Float?
  salaryMax          Float?
  employmentTypeId   Int
  benefits           String?
  companyDescription String?
  applicationDeadline DateTime?
  contactInfo        String?
  createdAt          DateTime         @default(now())
  updatedAt          DateTime         @updatedAt

  company       Company       @relation(fields: [companyId], references: [id])
  interviewFlow InterviewFlow @relation(fields: [interviewFlowId], references: [id])
  status        PositionStatus @relation(fields: [statusId], references: [id])
  employmentType EmploymentType @relation(fields: [employmentTypeId], references: [id])
  applications  Application[]

  @@index([companyId])       // Índice para búsquedas por empresa
  @@index([interviewFlowId]) // Índice para búsquedas por flujo de entrevistas
  @@index([title])           // Índice para búsquedas por título
  @@index([statusId])        // Índice para búsquedas por estado del puesto
}

model InterviewFlow {
  id           Int             @id @default(autoincrement())
  description  String
  createdAt    DateTime        @default(now())
  updatedAt    DateTime        @updatedAt
  positions    Position[]
  steps        InterviewStep[]

  @@index([description]) // Índice para búsquedas por descripción
}

model InterviewType {
  id          Int             @id @default(autoincrement())
  name        String          @unique
  description String?
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt
  steps       InterviewStep[]
}

model InterviewStep {
  id              Int           @id @default(autoincrement())
  interviewFlowId Int
  interviewTypeId Int
  name            String
  orderIndex      Int
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt

  interviewFlow InterviewFlow @relation(fields: [interviewFlowId], references: [id])
  interviewType InterviewType @relation(fields: [interviewTypeId], references: [id])
  interviews    Interview[]

  @@index([interviewFlowId]) // Índice para búsquedas por flujo de entrevistas
  @@index([interviewTypeId]) // Índice para búsquedas por tipo de entrevista
}

model Candidate {
  id        Int      @id @default(autoincrement())
  firstName String
  lastName  String
  email     String   @unique
  phone     String?
  address   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  applications Application[]

  @@index([lastName]) // Índice para búsquedas rápidas por apellido
}

model ApplicationStatus {
  id          Int         @id @default(autoincrement())
  name        String      @unique
  applications Application[]
}

model Application {
  id           Int       @id @default(autoincrement())
  positionId   Int
  candidateId  Int
  applicationDate DateTime @default(now())
  statusId     Int
  notes        String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  position   Position   @relation(fields: [positionId], references: [id])
  candidate  Candidate  @relation(fields: [candidateId], references: [id])
  status     ApplicationStatus @relation(fields: [statusId], references: [id])
  interviews Interview[]

  @@index([positionId])  // Índice para búsquedas rápidas por puesto
  @@index([candidateId]) // Índice para búsquedas rápidas por candidato
  @@index([statusId])    // Índice para búsquedas por estado
}

model InterviewResult {
  id         Int        @id @default(autoincrement())
  name       String     @unique
  interviews Interview[]
}

model Interview {
  id              Int           @id @default(autoincrement())
  applicationId   Int
  interviewStepId Int
  employeeId      Int
  interviewDate   DateTime
  resultId        Int?
  score           Int?          @default(0)
  notes           String?
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt

  application   Application   @relation(fields: [applicationId], references: [id])
  interviewStep InterviewStep @relation(fields: [interviewStepId], references: [id])
  employee      Employee      @relation(fields: [employeeId], references: [id])
  result        InterviewResult? @relation(fields: [resultId], references: [id])

  @@index([applicationId])   // Índice para búsquedas rápidas por aplicación
  @@index([interviewStepId]) // Índice para búsquedas rápidas por paso de entrevista
  @@index([employeeId])      // Índice para búsquedas rápidas por empleado
  @@index([interviewDate])   // Índice para búsquedas rápidas por fecha
}"
```

```markdown
vamor a realizar una migración desde cero, de lo que hay que el archivo #file:schema.prisma hacia la base de datos
```

En este punto ya se actualizó el schema.prisma, se realizó la migración y se pudo verificar el resultado de la migración a través de prisma studio.