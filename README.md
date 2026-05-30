# AMICALE-STAR Platform - Complete Architecture & Technical Documentation

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Architecture Overview](#system-architecture-overview)
3. [Backend Architecture (Spring Boot)](#backend-architecture-spring-boot)
4. [Frontend Architecture (Angular)](#frontend-architecture-angular)
5. [Technology Stack](#technology-stack)
6. [Design Patterns & Principles](#design-patterns--principles)
7. [Core Annotations Explained](#core-annotations-explained)
8. [Key Methods & Their Purpose](#key-methods--their-purpose)
9. [Jury Q&A Preparation](#jury-qa-preparation)

---

## Executive Summary

**AMICALE-STAR** is a comprehensive association management platform designed for student unions and organizations. The platform streamlines:

- **User Management**: Authentication, registration, role-based access control
- **Elections**: Electoral processes with candidacy, voting, and result publication
- **Surveys (Sondages)**: Democratic polling and feedback collection
- **Job Offers (Offres)**: Activity/job management with inscription system
- **Membership (Adhesion)**: Membership requests and fee management
- **Notifications**: Event-driven email and system notifications
- **Admin Dashboard**: Comprehensive management interface

The system follows **enterprise-grade architecture** principles with clear separation of concerns, event-driven communication, and role-based security.

---

## System Architecture Overview

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (Angular 21)                     │
│  ┌──────────┬──────────┬──────────┬──────────┐               │
│  │  Admin   │  Bureau  │ Adherent │  Public  │               │
│  │ Dashboard│Dashboard │Dashboard │  Login   │               │
│  └──────────┴──────────┴──────────┴──────────┘               │
│              Routes | Guards | Interceptors                  │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP/REST (JWT Authenticated)
┌──────────────────────▼──────────────────────────────────────┐
│              Backend (Spring Boot 3.2.5)                     │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ API Controllers (REST Endpoints)                     │    │
│  │ ├─ AuthController          ├─ SondageController     │    │
│  │ ├─ OffreController         ├─ AdherenceController   │    │
│  │ ├─ ElectionController      ├─ NotificationCtrl      │    │
│  │ └─ InscriptionController   └─ DashboardController   │    │
│  └────────────────┬──────────────────────────────────┬─┘    │
│                   │                                   │      │
│  ┌────────────────▼─────┐        ┌──────────────────▼────┐  │
│  │ Service Layer        │        │ Event Listeners       │  │
│  │ ├─ AuthService       │        │ ├─ ElectionListener   │  │
│  │ ├─ OffreService      │        │ ├─ InscriptionList.   │  │
│  │ ├─ ElectionService   │        │ └─ EmailFailureList.  │  │
│  │ ├─ InscriptionServ.  │        └──────────────────────┘  │
│  │ └─ NotificationServ. │                                   │
│  └────────────────┬─────┘        ┌──────────────────────┐   │
│                   │              │ Email Service        │   │
│                   │              │ (Async) @Async       │   │
│                   │              │ @Scheduled           │   │
│                   │              └──────────────────────┘   │
│  ┌────────────────▼──────────────────────────────────────┐  │
│  │ Data Access Layer (Repository)                         │  │
│  │ ├─ UserRepository      ├─ OffreRepository             │  │
│  │ ├─ ElectionRepository  ├─ InscriptionRepository       │  │
│  │ └─ SondageRepository   └─ AdhesionRepository          │  │
│  └────────────────┬──────────────────────────────────────┘  │
│                   │                                          │
│  ┌────────────────▼──────────────────────────────────────┐  │
│  │ Data Mapping Layer (Mapper Pattern)                    │  │
│  │ ├─ OffreMapper         ├─ ElectionMapper              │  │
│  │ └─ InscriptionMapper   └─ UserMapper                  │  │
│  └─────────────────────────────────────────────────────┬─┘  │
│                                                        │     │
└────────────────────────────────┬──────────────────────┘     │
                                 │
┌────────────────────────────────▼──────────────────────────────┐
│              Data Persistence Layer (JPA)                      │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  MySQL Database                          │ │
│  │  ├─ user (Parent for JOINED inheritance)                │ │
│  │  ├─ admin, membre_bureau, adherent (Child tables)       │ │
│  │  ├─ offre, inscription, election, sondage              │ │
│  │  └─ notification, pole, payment records                 │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Backend Architecture (Spring Boot)

### Package Structure & Responsibilities

```
tn.star.Pfe
├── config/                          # Configuration & Setup
│   ├── SecurityConfig.java          # JWT & Spring Security setup
│   ├── WebConfig.java               # CORS, interceptors, beans
│   └── DataInitializer.java         # Initial data seeding
│
├── controller/                      # REST API Endpoints
│   ├── auth/AuthController.java
│   ├── offre/OffreController.java
│   ├── election/ElectionController.java
│   ├── inscription/InscriptionController.java
│   ├── sondage/SondageController.java
│   ├── adhesion/AdhesionController.java
│   └── dashboard/DashboardController.java
│
├── service/                         # Business Logic Layer
│   ├── auth/
│   │   ├── IAuthService.java        # Interface (contract)
│   │   └── AuthService.java         # Implementation
│   ├── offre/
│   │   ├── IOffreService.java
│   │   └── OffreService.java        # Offre business logic
│   ├── election/
│   │   ├── IElectionService.java
│   │   └── ElectionService.java
│   ├── inscription/
│   │   ├── IInscriptionService.java
│   │   └── InscriptionService.java
│   ├── sondage/
│   ├── adhesion/
│   ├── notification/               # Async notification handling
│   ├── email/                       # Email sending (@Async)
│   └── dashboard/
│
├── repository/                      # Data Access Layer (JPA)
│   ├── user/UserRepository.java
│   ├── offer/OffreRepository.java
│   ├── election/ElectionRepository.java
│   ├── inscription/InscriptionRepository.java
│   ├── sondage/SondageRepository.java
│   └── adhesion/AdhesionRepository.java
│
├── entity/                          # JPA Entity Models
│   ├── user/
│   │   ├── User.java                # Abstract parent entity
│   │   ├── Admin.java               # Extends User
│   │   ├── MembreBureau.java        # Extends User
│   │   ├── Adherent.java            # Extends User
│   │   └── Pole.java                # Department/Team
│   ├── offre/
│   │   ├── Offre.java
│   │   └── OffreImage.java
│   ├── election/
│   │   ├── ElectionCall.java        # Election announcement
│   │   ├── CandidateApplication.java
│   │   ├── Position.java            # Candidate position
│   │   └── Vote.java
│   ├── inscription/
│   │   └── Inscription.java
│   ├── sondage/
│   │   ├── Sondage.java
│   │   ├── OptionSondage.java
│   │   └── VoteSondage.java
│   └── adhesion/
│       └── AdhesionDemande.java
│
├── event/                           # Event-Driven Architecture
│   ├── ElectionCallCreatedEvent.java
│   ├── ElectionPublishedEvent.java
│   ├── CandidacySubmittedEvent.java
│   ├── InscriptionStatusChangedEvent.java
│   ├── OffreCreatedEvent.java
│   ├── AdhesionDemandeEvent.java
│   ├── SondageOpenedEvent.java
│   ├── SondageClosedEvent.java
│   └── EmailFailedEvent.java
│
├── listener/                        # Event Handlers (@EventListener)
│   ├── ElectionEventListener.java   # Listens for election events
│   ├── InscriptionEventListener.java
│   ├── OffreEventListener.java
│   ├── AdhesionEventListener.java
│   └── SondageEventListener.java
│
├── dto/                             # Data Transfer Objects
│   ├── auth/
│   │   ├── login/LoginRequest.java
│   │   ├── create/CreateUserDto.java
│   │   └── update/UpdateUserDto.java
│   ├── offre/OffreRequest.java, OffreResponse.java
│   ├── election/ElectionDto.java
│   ├── inscription/InscriptionDto.java
│   └── notification/NotificationDto.java
│
├── mapper/                          # DTO ↔ Entity Mapping
│   ├── OffreMapper.java             # Maps Offre ↔ OffreResponse
│   ├── UserMapper.java
│   ├── ElectionMapper.java
│   └── InscriptionMapper.java
│
├── security/                        # Security & Authentication
│   ├── JwtUtils.java                # JWT generation/validation
│   ├── JwtAuthFilter.java           # Security filter for JWT
│   ├── UserPrincipal.java           # Principal implementation
│   └── CustomUserDetailsService.java
│
├── enums/                           # Constants & Enumerations
│   ├── Role.java                    # ADMIN, MEMBRE_BUREAU, ADHERENT
│   ├── OffreStatus.java
│   ├── InscriptionStatus.java
│   └── ElectionStatus.java
│
├── exceptions/                      # Custom Exceptions
│   ├── BadRequestException.java
│   ├── ResourceNotFoundException.java
│   ├── UnauthorizedException.java
│   └── ConflictException.java
│
└── PfeApplication.java              # Main entry point
```

### Key Design Patterns

#### 1. **Service-Repository Pattern**
```
Controller → Service (Business Logic) → Repository (Data Access) → Database
```

**Why?**
- **Separation of Concerns**: Each layer has a single responsibility
- **Testability**: Easy to mock repositories in unit tests
- **Reusability**: Services can be called from multiple controllers
- **Transaction Management**: Services handle @Transactional boundaries

#### 2. **Mapper Pattern** (Manual Mapping)
```
Entity → Mapper → DTO
DTO → Mapper → Entity
```

**Why?**
- **Decoupling**: Frontend doesn't depend on JPA entities
- **Transformation**: Can apply business logic during mapping
- **Security**: Avoid exposing sensitive entity fields
- **Flexibility**: Different DTOs for different endpoints

**Example from OffreMapper.java:**
```java
public OffreResponse toResponse(Offre offre) {
    OffreResponse res = new OffreResponse();
    res.setId(offre.getId());
    // Convert images to Base64 for JSON response
    if (offre.getImage() != null) {
        res.setImageBase64(Base64.getEncoder().encodeToString(offre.getImage()));
    }
    return res;
}
```

#### 3. **Event-Driven Architecture**
```
Business Operation → Publish Event → Event Listener → Async Handler (Email, Notification)
```

**Why?**
- **Decoupling**: Services don't directly call email/notification services
- **Scalability**: Events can be processed asynchronously
- **Resilience**: Failed emails don't crash the main transaction
- **Maintainability**: New features can listen to events without modifying existing code

**Example Flow:**
1. User applies for election candidate position
2. Service publishes `CandidacySubmittedEvent`
3. `ElectionEventListener` catches the event
4. Listener asynchronously sends confirmation email to candidate
5. Listener sends notification to admins

#### 4. **Entity Inheritance (JPA JOINED Strategy)**
```
┌─────────────────────┐
│  User (Abstract)    │ ← Base table with common fields
│ ├─ id, email, etc   │
└──────────┬──────────┘
      ┌────┴───────────┐
      │                │
  ┌───▼────┐      ┌───▼─────┐      ┌──────────┐
  │ Admin   │      │ Adherent │      │ Bureau   │
  └────────┘      └──────────┘      └──────────┘
```

**Why?**
- **Code Reuse**: Common fields (email, password) defined once
- **Polymorphism**: Query all users regardless of type
- **Type Safety**: Each role has specialized fields
- **Database Integrity**: Foreign key constraints work naturally

### Detailed Method Explanations

#### **AuthService.login()**
```java
@Transactional
public AuthResponse login(LoginRequest request) {
    // 1. Authenticate using Spring Security's AuthenticationManager
    Authentication auth = authManager.authenticate(
        new UsernamePasswordAuthenticationToken(
            request.email(),
            request.motDePasse()
        )
    );
    
    // 2. Extract authenticated user principal
    UserPrincipal principal = (UserPrincipal) auth.getPrincipal();
    
    // 3. Build JWT token and return response
    return buildAuthResponse(principal);
}
```

**Purpose**: Authenticate user via email & password, return JWT token

**Why This Implementation?**
- Uses Spring Security's built-in authentication
- BCryptPasswordEncoder handles password hashing securely
- Records login attempt for security auditing
- Returns JWT for stateless authentication

---

#### **OffreService.creer()** (Create Offer)
```java
public OffreResponse creer(OffreRequest req, MultipartFile image, 
                           List<MultipartFile> images, String createdByEmail) 
                           throws IOException {
    // 1. Validate user has permission
    User creator = userRepository.findByEmail(createdByEmail)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    
    // 2. Create entity from request DTO
    Offre offre = Offre.builder()
        .titre(req.titre())
        .description(req.description())
        .dateDebut(req.dateDebut())
        .dateFin(req.dateFin())
        .capaciteMax(req.capaciteMax())
        .statut(OffreStatus.DRAFT)  // Default status
        .createdBy(creator)
        .build();
    
    // 3. Handle image uploads
    if (image != null && !image.isEmpty()) {
        offre.setImage(image.getBytes());
        offre.setImageType(image.getContentType());
    }
    
    // 4. Save to database
    Offre saved = offreRepository.save(offre);
    
    // 5. Publish event for listeners (email notification, etc)
    applicationEventPublisher.publishEvent(new OffreCreatedEvent(saved));
    
    // 6. Map to response DTO and return
    return offreMapper.toResponse(saved);
}
```

**Purpose**: Create a new job offer/activity with image attachments

**Why This Implementation?**
- **Input Validation**: Checks file sizes before processing
- **Transaction Safety**: All operations succeed or none do
- **Event Publishing**: Notifies system of new offer (email, notification)
- **Image Handling**: Converts files to bytes for database storage
- **DTO Mapping**: Returns clean response without internal details

---

#### **InscriptionService.confirmerInscription()** (Approve Registration)
```java
@Transactional
public InscriptionResponse confirmerInscription(Long id) {
    Inscription inscription = inscriptionRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Inscription not found"));
    
    // 1. Check current status is PENDING
    if (!inscription.getStatut().equals(InscriptionStatus.PENDING)) {
        throw new BadRequestException("Can only confirm pending inscriptions");
    }
    
    // 2. Check capacity hasn't been exceeded
    Offre offre = inscription.getOffre();
    if (offre.getPlacesRestantes() <= 0) {
        throw new ConflictException("No more available slots");
    }
    
    // 3. Update inscription status
    inscription.setStatut(InscriptionStatus.CONFIRMED);
    inscription.setConfirmedAt(LocalDateTime.now());
    
    // 4. Decrement remaining slots
    offre.setPlacesRestantes(offre.getPlacesRestantes() - 1);
    
    // 5. Save and publish event
    Inscription saved = inscriptionRepository.save(inscription);
    applicationEventPublisher.publishEvent(
        new InscriptionStatusChangedEvent(saved, InscriptionStatus.CONFIRMED)
    );
    
    return mapper.toResponse(saved);
}
```

**Purpose**: Approve a user's registration for an offer

**Why This Implementation?**
- **Concurrency Safe**: @Transactional ensures atomic updates
- **Business Rules**: Validates status transitions
- **Capacity Management**: Prevents overbooking
- **Event-Driven**: Triggers email confirmation to user
- **Audit Trail**: Records confirmation timestamp

---

#### **ElectionEventListener.onElectionCallCreated()** (@EventListener with @Async)
```java
@Async              // ← Runs in separate thread pool
@EventListener      // ← Triggered by ElectionCallCreatedEvent
public void onElectionCallCreated(ElectionCallCreatedEvent event) {
    log.info("ElectionCall created: id={}", event.call().getId());
    
    // 1. Find all administrators
    List<User> admins = userRepository.findByRole(Role.ADMIN);
    
    // 2. Send email to each admin asynchronously
    for (User admin : admins) {
        try {
            emailService.sendElectionCallCreatedToAdmins(
                admin.getEmail(),
                event.call().getTitre(),
                event.call().getDescription()
            );
        } catch (Exception e) {
            // Log error but don't crash
            log.error("Failed to send email to admin: {}", admin.getEmail(), e);
            // Could publish EmailFailedEvent here for retry mechanism
        }
    }
}
```

**Purpose**: Send async notifications when election announcement is created

**Why This Implementation?**
- **@Async**: Prevents email sending from blocking the main request
- **@EventListener**: Automatically triggered by Spring when event is published
- **Error Handling**: Failed emails don't crash the system
- **Decoupling**: Election service doesn't know about email logic
- **Scalability**: Can add more listeners without modifying election service

---

## Frontend Architecture (Angular)

### Project Structure

```
frontend-amicale/src/app
├── core/                            # Singleton services & guards
│   ├── guards/
│   │   ├── auth.guard.ts           # Protects authenticated routes
│   │   ├── admin.guard.ts          # Protects admin-only routes
│   │   ├── bureau.guard.ts         # Protects bureau member routes
│   │   └── admin-or-bureau.guard.ts # Dual role protection
│   │
│   └── services/
│       ├── auth.service.ts         # Authentication & token management
│       ├── election.service.ts     # Election data API calls
│       ├── sondage.service.ts      # Survey API calls
│       └── notification.service.ts # Notification handling
│
├── shared/                          # Reusable components & utilities
│   ├── components/
│   │   ├── user-profil/
│   │   └── shared-components.ts
│   ├── pipes/
│   │   └── custom pipes for data formatting
│   └── utils/
│       └── helper functions
│
├── auth/                            # Authentication feature
│   ├── login/login.component.ts
│   ├── register/register.component.ts
│   ├── forgot-password/
│   └── change-password/
│
├── admin/                           # Admin-only features
│   ├── layout/admin-layout.component.ts
│   ├── dashboard/admin-dashboard.component.ts
│   ├── users/users.component.ts
│   ├── offres/offres.component.ts
│   ├── creer-offre/creer-offre.component.ts
│   ├── calendrier/
│   ├── activites/
│   └── guards/admin.guard.ts
│
├── bureau/                          # Bureau member features
│   ├── layout/bureau-layout.component.ts
│   ├── dashboard/bureau-dashboard.component.ts
│   ├── offres/bureau-offres.component.ts
│   ├── sondages/
│   │   ├── creer-sondage/
│   │   └── elections/
│   │       ├── election-list/
│   │       ├── election-apply/
│   │       ├── election-voting/
│   │       ├── election-results/
│   │       └── election-call-admin/
│   ├── inscriptions/
│   ├── cotisations/
│   ├── statistiques/
│   └── guards/bureau.guard.ts
│
├── adherent/                        # Member/Adherent features
│   ├── layout/adherent-layout.component.ts
│   ├── dashboard/adherent-dashboard.component.ts
│   ├── offres/adherent-offres.component.ts
│   ├── offre-detail/
│   ├── inscriptions/
│   ├── sondages/
│   ├── cotisation/
│   ├── annonces/
│   ├── calendrier/
│   └── profil/
│
├── home/home.component.ts           # Home/landing page
├── partners/                        # Partner listings
├── activities/                      # Activities management
└── app.routes.ts                   # Route definitions

```

### Frontend Design Patterns

#### 1. **Route Guards** - Role-Based Access Control
```typescript
// auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  if (authService.isAuthenticated()) {
    return true;
  }
  return inject(Router).createUrlTree(['/login']);
};

// admin.guard.ts  
export const adminGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  if (authService.isAdmin()) {
    return true;
  }
  return inject(Router).createUrlTree(['/unauthorized']);
};
```

**Purpose**: Prevent unauthorized access to protected routes

**Implementation:**
```typescript
{
  path: 'admin',
  canActivate: [adminGuard],  // ← Only ADMIN role can access
  loadComponent: () => import('./admin/layout').then(m => m.AdminLayoutComponent),
  children: [...]
}
```

#### 2. **Angular Signals** - Reactive State Management
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  // Signal: reactive primitive
  currentUser = signal<{ email: string; role: string } | null>(null);
  firstLogin = signal(false);
  photoUrl = signal<string | null>(null);
  
  // Computed: derived from signals, auto-updates
  isAdmin = computed(() => this.currentUser()?.role === 'ADMIN');
  isBureau = computed(() => this.currentUser()?.role === 'MEMBRE_BUREAU');
  isAdherent = computed(() => this.currentUser()?.role === 'ADHERENT');
  
  login(email: string, motDePasse: string) {
    return this.http
      .post<AuthResponse>(`${this.API}/login`, { email, motDePasse })
      .pipe(
        tap((res) => {
          // Update signal → automatically updates all dependents
          this.currentUser.set({
            email: payload.sub,
            role: res.role,
            prenom: payload.prenom,
            nom: payload.nom,
          });
          this.firstLogin.set(res.firstLogin);
        })
      );
  }
}
```

**In Template:**
```html
<div *ngIf="authService.isAdmin(); else notAdmin">
  <a href="/admin">Go to Admin Panel</a>
</div>
<ng-template #notAdmin>
  <p>You don't have admin access</p>
</ng-template>
```

**Why?**
- **Reactive**: Automatic change detection when signals update
- **Performance**: Only affected components re-render
- **Simple API**: No Observable complexity for basic state

#### 3. **Standalone Components** - Modern Angular Pattern
```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { RouterLink } from '@angular/router';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-admin-dashboard',
  standalone: true,  // ← No NgModule required
  imports: [CommonModule, RouterLink, HttpClientModule],  // ← Explicit dependencies
  template: `<h1>Admin Dashboard</h1>`,
})
export class AdminDashboardComponent {
  private http = inject(HttpClient);  // ← Dependency injection via inject()
  
  ngOnInit() {
    this.http.get('/api/admin/stats').subscribe(data => {
      // Handle data
    });
  }
}
```

**Why?**
- **No NgModule boilerplate**: Simpler code organization
- **Tree-shakeable**: Unused components removed from bundle
- **Clearer dependencies**: Explicit imports per component
- **Better Code Splitting**: Lazy loading works naturally

#### 4. **Lazy Loading Route Components**
```typescript
{
  path: 'admin',
  canActivate: [adminGuard],
  loadComponent: () => import('./admin/layout/admin-layout.component')
    .then(m => m.AdminLayoutComponent),  // ← Component loaded on-demand
  children: [
    {
      path: 'dashboard',
      loadComponent: () => import('./admin/dashboard/dashboard.component')
        .then(m => m.AdminDashboardComponent)  // ← Lazy loaded
    }
  ]
}
```

**Purpose**: Only download component code when user navigates to it

**Benefits:**
- **Faster Initial Load**: Smaller JavaScript bundle
- **Bandwidth Savings**: Users only download what they use
- **Better UX**: Quick initial page load

#### 5. **Async Pipe** - Automatic Subscription Management
```typescript
// Component
export class OffreListComponent {
  offres$ = this.offreService.getAllOffres();  // ← Returns Observable
  
  constructor(private offreService: OffreService) {}
}

// Template
<div *ngFor="let offre of (offres$ | async)">
  <h3>{{ offre.titre }}</h3>
  <p>{{ offre.description }}</p>
</div>
```

**Why?**
- **Automatic Unsubscribe**: Prevents memory leaks
- **Cleaner Code**: No subscription management needed
- **Change Detection**: Works with OnPush strategy

---

## Technology Stack

### Backend Dependencies
```xml
<!-- Spring Boot & Framework -->
<spring-boot-starter-web>           <!-- REST API -->
<spring-boot-starter-data-jpa>      <!-- ORM & Database -->
<spring-boot-starter-security>      <!-- Authentication & Authorization -->
<spring-boot-starter-validation>    <!-- Input validation -->
<spring-boot-starter-mail>          <!-- Email sending -->

<!-- JWT Authentication -->
<jjwt-api>                          <!-- JWT token generation -->
<jjwt-impl>
<jjwt-jackson>

<!-- Database -->
<mysql-connector-j>                 <!-- MySQL driver -->
<h2database>                        <!-- In-memory for testing -->

<!-- Utilities -->
<lombok>                            <!-- @Getter, @Setter, @Builder -->
<mapstruct>                         <!-- Entity ↔ DTO mapping -->
<spring-dotenv>                     <!-- .env file support -->

<!-- Testing -->
<spring-boot-starter-test>
<spring-security-test>
```

### Frontend Dependencies
```json
{
  "@angular/core": "^21.2.0",       // Core Angular framework
  "@angular/common": "^21.2.0",     // Common utilities
  "@angular/forms": "^21.2.0",      // Reactive & Template Forms
  "@angular/router": "^21.2.0",     // SPA routing
  "@angular/platform-browser": "^21.2.0",  // Browser rendering
  "rxjs": "~7.8.0",                 // Reactive Extensions (Observables)
  "lucide-angular": "^1.0.0"        // Icon library
}
```

### Development Tools
```json
{
  "@angular/cli": "^21.2.6",        // CLI for ng commands
  "typescript": "~5.9.2",           // TypeScript compiler
  "vitest": "4.0.8",                // Unit testing framework
  "prettier": "^3.8.1"              // Code formatting
}
```

---

## Design Patterns & Principles

### 1. **SOLID Principles Implementation**

#### **S - Single Responsibility Principle**
```java
// ❌ BAD: Multiple responsibilities
public class OffreService {
    public void createOffre(...) { }
    public void sendEmailToUsers() { }
    public void logToDatabase() { }
}

// ✅ GOOD: Single responsibility
public class OffreService {
    public void createOffre(...) { }  // Only offer creation
}

public class EmailService {
    public void sendEmailToUsers() { }  // Only email
}

public class AuditLogger {
    public void logToDatabase() { }  // Only logging
}
```

#### **O - Open/Closed Principle**
```java
// ✅ Open for extension via interfaces
public interface IEmailService {
    void sendEmail(String to, String subject, String body);
}

public class SmtpEmailService implements IEmailService {
    @Override
    public void sendEmail(String to, String subject, String body) { }
}

public class MockEmailService implements IEmailService {
    @Override
    public void sendEmail(String to, String subject, String body) { }
}

// Extend without modifying existing code
```

#### **L - Liskov Substitution Principle**
```java
// ✅ Subtypes must be substitutable
User admin = new Admin();  // Can use as User
User member = new MembreBureau();  // Can use as User
User adherent = new Adherent();  // Can use as User

// All can be queried uniformly
List<User> allUsers = userRepository.findAll();
for (User user : allUsers) {
    user.getRole();  // Works for all subclasses
}
```

#### **I - Interface Segregation Principle**
```java
// ❌ BAD: Fat interface
public interface UserRepository {
    void create(User user);
    void update(User user);
    void delete(User user);
    void sendEmail(String email);  // ← Not a data access concern
    void generateReport();          // ← Not a data access concern
}

// ✅ GOOD: Segregated interfaces
public interface IUserRepository {
    void create(User user);
    void update(User user);
    void delete(User user);
}

public interface IEmailService {
    void sendEmail(String email);
}

public interface IReportGenerator {
    void generateReport();
}
```

#### **D - Dependency Inversion Principle**
```java
// ❌ BAD: Depends on concrete class
public class OffreService {
    private EmailService emailService = new EmailService();  // Hard dependency
    
    public void createOffre(OffreRequest req) {
        emailService.send("...");  // Tightly coupled
    }
}

// ✅ GOOD: Depends on abstraction
public class OffreService {
    private final IEmailService emailService;  // Interface
    
    @Autowired
    public OffreService(IEmailService emailService) {
        this.emailService = emailService;  // Injected dependency
    }
    
    public void createOffre(OffreRequest req) {
        emailService.send("...");  // Loosely coupled
    }
}

// Can inject different implementations:
// - SmtpEmailService (production)
// - MockEmailService (testing)
// - SendgridEmailService (another provider)
```

### 2. **Design Patterns Used**

#### **Repository Pattern**
```
Interface/Contract ← Implementation
IUserRepository ← UserRepository
  ↓
  Data Access Logic
  (SQL queries via Spring Data JPA)
```

**Why?**
- Abstracts database details
- Easy to test with mock implementations
- Can switch database without changing service layer

#### **Builder Pattern** (via Lombok @Builder)
```java
// ❌ Without Builder
User user = new User();
user.setId(1L);
user.setEmail("user@example.com");
user.setNom("Zakraoui");
user.setPrenom("Melek");
user.setRole(Role.ADMIN);

// ✅ With Builder
User user = User.builder()
    .id(1L)
    .email("user@example.com")
    .nom("Zakraoui")
    .prenom("Melek")
    .role(Role.ADMIN)
    .build();
```

#### **Decorator Pattern** (Spring Annotations)
```java
@Service                    // ← Decorator: marks as service bean
@Transactional              // ← Decorator: auto transaction management
@RequiredArgsConstructor    // ← Decorator: auto constructor generation
public class OffreService {
    
    @Async                  // ← Decorator: run in thread pool
    @EventListener          // ← Decorator: auto event subscription
    public void handleEvent(MyEvent event) { }
}
```

#### **Observer Pattern** (Event-Driven)
```
Publisher (Service)
    ↓ publishes
Event (e.g., InscriptionStatusChangedEvent)
    ↓ subscribed by
Observer/Listener (e.g., InscriptionEventListener)
    ↓ handles
Reaction (send email, update cache, log)
```

---

## Core Annotations Explained

### Backend Annotations

#### **Spring Framework Core**

| Annotation | Location | Purpose | When to Use |
|-----------|----------|---------|------------|
| `@SpringBootApplication` | Main Class | Marks entry point, enables auto-config | Once per application |
| `@Configuration` | Config Classes | Defines Spring beans and configuration | Setup, database config |
| `@Bean` | Methods in @Config | Registers object as Spring bean | Custom beans, third-party classes |
| `@Autowired` | Fields/Constructor | Dependency injection | Wire dependencies (prefer constructor) |
| `@Component` | Classes | Generic Spring-managed component | Utility classes, helpers |
| `@Service` | Service Classes | Marks business logic layer | Service implementations |
| `@Repository` | DAO Classes | Marks data access layer, enables exception translation | JPA repositories |
| `@Controller` / `@RestController` | API Classes | Marks as web handler | API controllers |

#### **Dependency Injection**

```java
// ❌ Field injection (not recommended)
@Service
public class OffreService {
    @Autowired
    private IOffreService offreService;
}

// ✅ Constructor injection (recommended)
@Service
@RequiredArgsConstructor  // ← Lombok: generates constructor
public class OffreService {
    private final IOffreService offreService;  // ← Final & immutable
}

// ✅ Setter injection (less common)
@Service
public class OffreService {
    private IOffreService offreService;
    
    @Autowired
    public void setOffreService(IOffreService service) {
        this.offreService = service;
    }
}
```

**Why Constructor Injection?**
- Immutable (`final` fields)
- Dependencies explicit in signature
- Easier to test (can pass mocks)
- Fails fast if dependency missing

#### **Web Request Handling**

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@RequestMapping` | Maps HTTP requests to handler | `@RequestMapping("/api/offres")` |
| `@GetMapping` | Maps GET requests | `@GetMapping("/{id}")` |
| `@PostMapping` | Maps POST requests | `@PostMapping("/creer")` |
| `@PutMapping` | Maps PUT requests | `@PutMapping("/{id}")` |
| `@PatchMapping` | Maps PATCH requests | `@PatchMapping("/{id}/status")` |
| `@DeleteMapping` | Maps DELETE requests | `@DeleteMapping("/{id}")` |
| `@PathVariable` | Extracts URL path variable | `@PathVariable Long id` |
| `@RequestParam` | Extracts query parameter | `@RequestParam String search` |
| `@RequestBody` | Deserializes JSON body | `@RequestBody OffreRequest req` |
| `@ResponseBody` | Serializes return as JSON | Implicit in @RestController |
| `@RequestPart` | Multipart file upload | `@RequestPart MultipartFile image` |

```java
@RestController
@RequestMapping("/api/offres")
public class OffreController {
    
    // GET /api/offres/123
    @GetMapping("/{id}")
    public ResponseEntity<OffreResponse> getOffre(
        @PathVariable Long id  // ← Extract from URL
    ) { }
    
    // POST /api/offres/search?titre=Java
    @PostMapping("/search")
    public ResponseEntity<List<OffreResponse>> search(
        @RequestParam String titre  // ← Extract from query string
    ) { }
    
    // POST /api/offres/creer
    // Request body: { "titre": "...", "description": "..." }
    @PostMapping("/creer")
    public ResponseEntity<OffreResponse> create(
        @RequestBody OffreRequest req  // ← Extract from JSON body
    ) { }
    
    // POST /api/offres/creer with file upload
    @PostMapping(value = "/creer", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<OffreResponse> createWithImage(
        @RequestPart("req") OffreRequest req,
        @RequestPart(value = "image", required = false) MultipartFile image
    ) { }
}
```

#### **Data Access & ORM**

| Annotation | Location | Purpose |
|-----------|----------|---------|
| `@Entity` | Class | JPA entity representing database table |
| `@Table` | Class | Specifies table name |
| `@Id` | Field | Primary key |
| `@GeneratedValue` | Field | Auto-generate ID (auto_increment, UUID, etc) |
| `@Column` | Field | Column configuration (nullable, unique, length) |
| `@Lob` | Field | Large object (BLOB/CLOB for images, files) |
| `@OneToMany` | Field | One-to-many relationship |
| `@ManyToOne` | Field | Many-to-one relationship |
| `@JoinColumn` | Field | Foreign key column name |
| `@Inheritance` | Class | Strategy for inherited entities |
| `@DiscriminatorColumn` | Class | Column distinguishing subclasses |
| `@CreationTimestamp` | Field | Auto-set to creation time |
| `@UpdateTimestamp` | Field | Auto-set to update time |

```java
@Entity
@Table(name = "offre")
@Getter @Setter
public class Offre {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;  // Auto-increment ID
    
    @Column(nullable = false, length = 255)
    private String titre;
    
    @Lob  // Large text field (up to 4GB)
    @Column(columnDefinition = "LONGTEXT")
    private String description;
    
    @CreationTimestamp
    private LocalDateTime createdAt;  // Auto-set on insert
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;  // Auto-set on update
    
    @ManyToOne
    @JoinColumn(name = "pole_id")
    private Pole pole;  // Foreign key to pole table
    
    @OneToMany(mappedBy = "offre", cascade = CascadeType.ALL)
    private List<Inscription> inscriptions;  // Inverse side
}
```

#### **Security & Authorization**

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@EnableWebSecurity` | Enables Spring Security | Configuration class |
| `@EnableMethodSecurity` | Enables method-level authorization | Configuration class |
| `@PreAuthorize` | Method-level access control | Before method execution |
| `@PostAuthorize` | Post-execution check | After method execution |
| `@AuthenticationPrincipal` | Inject authenticated user | In controller methods |
| `@Secured` | Role-based access | Alternative to @PreAuthorize |

```java
@RestController
@RequestMapping("/api/offres")
public class OffreController {
    
    // Anyone can view public offers
    @GetMapping
    public ResponseEntity<List<OffreResponse>> list() { }
    
    // Only ADMIN or MEMBRE_BUREAU can create
    @PostMapping("/creer")
    @PreAuthorize("hasAnyRole('ADMIN', 'MEMBRE_BUREAU')")
    public ResponseEntity<OffreResponse> create(
        @RequestBody OffreRequest req,
        @AuthenticationPrincipal UserDetails user  // ← Get current user
    ) { }
    
    // Multiple conditions
    @PatchMapping("/{id}/publish")
    @PreAuthorize("hasRole('ADMIN') and #offre.createdBy.email == authentication.name")
    public ResponseEntity<OffreResponse> publish(
        @PathVariable Long id
    ) { }
}
```

#### **Validation**

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Valid` | Trigger validation | `@Valid @RequestBody OffreRequest` |
| `@NotNull` | Field must not be null | `private String titre;` |
| `@NotBlank` | String must not be empty | `@NotBlank private String description;` |
| `@Email` | Valid email format | `@Email private String email;` |
| `@Min` / `@Max` | Numeric range | `@Min(0) @Max(100)` |
| `@Size` | String/Collection size | `@Size(min=1, max=10)` |
| `@Pattern` | Regex matching | `@Pattern(regexp="...")` |
| `@Positive` | Must be > 0 | `@Positive private int count;` |

```java
@RequestBody
@Valid  // ← Trigger validation
public OffreRequest {
    @NotBlank(message = "Title is required")
    private String titre;
    
    @Email(regexp = "^[a-zA-Z0-9_!#$%&'*+/=?`{|}~^.-]+@...")
    private String email;
    
    @Positive(message = "Price must be positive")
    private BigDecimal prix;
    
    @Size(min=1, max=5, message = "Need 1-5 images")
    private List<String> imageUrls;
}
```

#### **Transaction Management**

| Annotation | Purpose | Configuration |
|-----------|---------|-----------------|
| `@Transactional` | Manages database transaction | Method or class level |
| `readOnly=true` | Optimization: read-only transaction | `@Transactional(readOnly=true)` |
| `propagation=REQUIRED` | Create new or use existing | Default |
| `rollbackFor` | Which exceptions cause rollback | `@Transactional(rollbackFor=Exception.class)` |

```java
@Service
public class InscriptionService {
    
    // ✅ Write operation: needs transaction
    @Transactional
    public void confirmer(Long id) {
        inscription.setStatut(CONFIRMED);
        inscription.setConfirmedAt(now);
        inscriptionRepository.save(inscription);  // Committed
    }
    
    // ✅ Read-only optimization
    @Transactional(readOnly=true)
    public List<Inscription> listerPour(Long offreId) {
        return inscriptionRepository.findByOffreId(offreId);  // No commit needed
    }
    
    // ✅ Custom rollback
    @Transactional(rollbackFor = BusinessException.class)
    public void complexOperation() {
        // If BusinessException thrown: ROLLBACK
        // If RuntimeException thrown: ROLLBACK (by default)
        // If checked exception thrown: COMMIT (by default)
    }
}
```

#### **Asynchronous Processing**

| Annotation | Purpose | Setup |
|-----------|---------|-------|
| `@EnableAsync` | Enables @Async | In @Configuration or Main class |
| `@Async` | Run method in thread pool | Service methods |
| `@EnableScheduling` | Enables @Scheduled | In @Configuration or Main class |
| `@Scheduled` | Run periodically | Service methods |

```java
// PfeApplication.java
@SpringBootApplication
@EnableAsync           // ← Enable async methods
@EnableScheduling      // ← Enable scheduled tasks
public class PfeApplication { }

// Service.java
@Service
@RequiredArgsConstructor
public class EmailService {
    
    // Runs in background thread
    // Non-blocking return to caller
    @Async
    public void sendEmailAsync(String to, String subject, String body) {
        // Slow I/O operation doesn't block request
        mailService.send(to, subject, body);
    }
    
    // Runs every day at 2 AM
    @Scheduled(cron = "0 0 2 * * *")
    public void dailyEmailReport() {
        // Generate and send daily reports
    }
    
    // Runs every 30 minutes
    @Scheduled(fixedDelay = 1800000)  // milliseconds
    public void periodicCleanup() {
        // Cleanup old records
    }
}
```

#### **Event-Driven Architecture**

| Annotation | Purpose | Context |
|-----------|---------|---------|
| `@EventListener` | Listen for application events | Spring events |
| `@TransactionalEventListener` | Listen after transaction commit | Consistency |
| `@Async` with `@EventListener` | Async event processing | Background tasks |

```java
// Publishing event
@Service
public class OffreService {
    @Autowired
    private ApplicationEventPublisher publisher;
    
    public void createOffre(OffreRequest req) {
        Offre offre = offreRepository.save(...);
        
        // Publish event
        publisher.publishEvent(new OffreCreatedEvent(offre));
    }
}

// Listening to event
@Component
@RequiredArgsConstructor
public class OffreEventListener {
    
    @Async                  // ← Run in background
    @EventListener          // ← Listen for event
    public void onOffreCreated(OffreCreatedEvent event) {
        log.info("Offre created: {}", event.offre().getTitre());
        // Send emails, update cache, etc.
    }
    
    @TransactionalEventListener  // ← After current transaction commits
    public void onOffreCreatedTx(OffreCreatedEvent event) {
        // Only runs if service's transaction succeeded
    }
}
```

#### **Lombok Annotations** (Reduce Boilerplate)

| Annotation | Purpose | Generates |
|-----------|---------|----------|
| `@Getter` / `@Setter` | Auto getters/setters | `public String getEmail() {}` |
| `@AllArgsConstructor` | Constructor with all fields | `public User(Long id, String email, ...)` |
| `@NoArgsConstructor` | No-argument constructor | `public User()` |
| `@RequiredArgsConstructor` | Constructor for final fields | Dependency injection |
| `@Builder` | Builder pattern | `User.builder().email("...").build()` |
| `@Slf4j` | Logger instance | `private static final Logger log = ...` |
| `@Data` | @Getter, @Setter, @ToString, @EqualsAndHashCode | All together |

```java
// ❌ Without Lombok (40 lines)
public class User {
    private Long id;
    private String email;
    private String nom;
    
    public User() {}
    public User(Long id, String email, String nom) {
        this.id = id;
        this.email = email;
        this.nom = nom;
    }
    
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getNom() { return nom; }
    public void setNom(String nom) { this.nom = nom; }
    
    @Override
    public String toString() { return "User{" + "id=" + id + ... + '}'; }
    
    @Override
    public boolean equals(Object o) { ... }
}

// ✅ With Lombok (5 lines)
@Entity
@Data  // Generates getters, setters, toString, equals, hashCode
@NoArgsConstructor  // No-arg constructor
@AllArgsConstructor // All-args constructor
public class User {
    private Long id;
    private String email;
    private String nom;
}
```

#### **MapStruct Annotations** (DTO Mapping)

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Mapper` | Interface is a mapper component | `@Mapper(componentModel = "spring")` |
| `@Mapping` | Field mapping configuration | `@Mapping(source = "offre.id", target = "offreId")` |

```java
@Mapper(componentModel = "spring")  // ← Registered as Spring bean
public interface OffreMapper {
    
    // Auto-map: matching field names mapped automatically
    OffreResponse toResponse(Offre offre);
    
    // Custom mapping
    @Mapping(source = "offre.pole.nom", target = "poleNom")
    @Mapping(source = "offre.createdBy.email", target = "createdByEmail")
    @Mapping(target = "imageBase64", expression = "java(encodeImage(offre.getImage()))")
    OffreResponse toResponseWithPole(Offre offre);
    
    // Helper method
    default String encodeImage(byte[] image) {
        return Base64.getEncoder().encodeToString(image);
    }
}
```

### Frontend Annotations (TypeScript Decorators & Angular)

#### **Component Decorators**

```typescript
@Component({
  selector: 'app-offre-list',           // ← Custom HTML tag
  standalone: true,                      // ← No NgModule needed (modern)
  imports: [CommonModule, HttpClient],   // ← Explicit dependencies
  template: `<h1>{{ title }}</h1>`,     // ← Inline template
  // OR
  templateUrl: './offre-list.component.html',  // ← External template
  styles: [`h1 { color: blue; }`],      // ← Inline styles
  // OR
  styleUrls: ['./offre-list.component.css'],   // ← External styles
  // Change detection strategy
  changeDetection: ChangeDetectionStrategy.OnPush  // ← Performance optimization
})
export class OffreListComponent { }
```

#### **Service Decorators**

```typescript
@Injectable({
  providedIn: 'root'  // ← Available app-wide (singleton)
})
export class AuthService {
  // Single instance shared across app
  private currentUser = signal<User | null>(null);
}

// OR provide at module level
@Injectable({
  providedIn: FeatureModule  // ← Available only in FeatureModule
})
export class FeatureService { }
```

#### **Directive Decorators**

```typescript
// Structural directive (changes DOM)
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  constructor(el: ElementRef) {
    el.nativeElement.style.backgroundColor = 'yellow';
  }
}

// Usage in template
<h1 [appHighlight]>Highlighted text</h1>

// Attribute directive
@Directive({
  selector: '[appTooltip]',
  host: {
    '(mouseenter)': 'show()',
    '(mouseleave)': 'hide()'
  }
})
export class TooltipDirective {
  @Input() appTooltip: string = '';  // ← Input binding
  
  show() { /* Show tooltip */ }
  hide() { /* Hide tooltip */ }
}
```

#### **Property Decorators**

```typescript
@Component({
  selector: 'app-card',
  template: `<div class="card">{{ content }}</div>`
})
export class CardComponent {
  // Input: receive value from parent
  @Input() content: string = '';
  @Input() title?: string;
  
  // Output: send events to parent
  @Output() onClick = new EventEmitter<void>();
  
  // View child: access child component
  @ViewChild(InputComponent) input!: InputComponent;
  
  // Content child: access ng-content
  @ContentChild(HeaderDirective) header!: HeaderDirective;
}

// Parent usage
<app-card
  [content]="'Hello'"      // ← Input binding
  [title]="cardTitle"      // ← Property binding
  (onClick)="handleClick()" // ← Event binding
>
  <app-header></app-header>
</app-card>
```

#### **Guard Functions**

```typescript
// CanActivate: control route access
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  
  if (authService.isAuthenticated()) {
    return true;  // Allow navigation
  }
  
  return inject(Router).createUrlTree(['/login']);  // Redirect
};

// CanDeactivate: prevent unsaved changes
export const unsavedChangesGuard: CanDeactivateFn<CanComponentDeactivate> = 
  (component) => {
    return component?.canDeactivate() ?? true;
  };

// Route configuration
{
  path: 'admin',
  canActivate: [authGuard, adminGuard],      // ← Multiple guards
  canDeactivate: [unsavedChangesGuard],      // ← Deactivation guard
  loadComponent: () => import('...').then(m => m.AdminComponent)
}
```

---

## Key Methods & Their Purpose

### Authentication Flow

**Backend:**
```java
// 1. User submits credentials
POST /api/auth/login
{
  "email": "user@example.com",
  "motDePasse": "password123"
}

// AuthController.java
@PostMapping("/login")
public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest req) {
    return ResponseEntity.ok(authService.login(req));
}

// 2. AuthService validates credentials
@Service
public class AuthService {
    @Transactional
    public AuthResponse login(LoginRequest request) {
        // Authenticate using Spring Security
        Authentication auth = authManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.email(),
                request.motDePasse()
            )
        );
        
        UserPrincipal principal = (UserPrincipal) auth.getPrincipal();
        log.info("User logged in: {}", principal.getUsername());
        
        // Generate JWT token
        return buildAuthResponse(principal);
    }
    
    private AuthResponse buildAuthResponse(UserPrincipal principal) {
        String role = principal.getAuthorities().stream()
            .findFirst()
            .map(GrantedAuthority::getAuthority)
            .orElse("ROLE_ADHERENT");
        
        return new AuthResponse(
            jwtUtils.generateAccessToken(principal),  // JWT token
            role.replace("ROLE_", ""),
            principal.isFirstLogin()
        );
    }
}

// 3. Return JWT token
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "role": "ADMIN",
  "firstLogin": false
}
```

**Frontend:**
```typescript
// 1. User submits login form
this.authService.login(email, password).subscribe({
  next: (response) => {
    // Store JWT token in localStorage
    localStorage.setItem('access_token', response.accessToken);
    
    // Store user info in signal
    this.currentUser.set({ email, role: response.role });
    
    // Navigate based on role
    if (response.role === 'ADMIN') {
      this.router.navigate(['/admin']);
    }
  },
  error: (err) => {
    this.error.set('Invalid credentials');
  }
});

// 2. Subsequent requests include JWT token
// HttpClient interceptor automatically adds:
// Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 3. Backend JwtAuthFilter validates token
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                  HttpServletResponse response,
                                  FilterChain filterChain) {
        // Extract JWT from Authorization header
        String token = getTokenFromRequest(request);
        
        if (token != null && jwtUtils.isValidToken(token)) {
            // Extract user info from JWT claims
            String email = jwtUtils.getEmailFromToken(token);
            User user = userRepository.findByEmail(email).orElse(null);
            
            if (user != null) {
                // Create authenticated principal
                UserPrincipal principal = new UserPrincipal(user);
                Authentication authentication = new UsernamePasswordAuthenticationToken(
                    principal, null, principal.getAuthorities()
                );
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

### Offer Management Flow

```
┌─────────────────────────────────────────────────────────┐
│ User (Bureau Member) Creates an Offer                   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ POST /api/offres/creer                                  │
│ - Title, description, dates, capacity, price           │
│ - Cover image (1 MB max), additional images             │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ OffreController.create()                                │
│ 1. Validate user has MEMBRE_BUREAU or ADMIN role        │
│ 2. Validate file sizes                                  │
│ 3. Call offreService.creer()                            │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ OffreService.creer()                                    │
│ 1. Build Offre entity from request DTO                  │
│ 2. Convert image files to bytes                         │
│ 3. Save to database via offreRepository.save()          │
│ 4. Publish OffreCreatedEvent                            │
│ 5. Return mapped OffreResponse DTO                      │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ OffreEventListener listens for OffreCreatedEvent        │
│ @Async @EventListener onOffreCreated()                  │
│ 1. Find all subscribers to this offer type              │
│ 2. Send async emails to all subscribers                 │
│ 3. Update notification feed                             │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│ OffreResponse returned to frontend                      │
│ {                                                        │
│   "id": 123,                                            │
│   "titre": "Summer Internship",                         │
│   "description": "...",                                 │
│   "imageBase64": "data:image/jpeg;base64,/9j/4AA...",  │
│   "status": "PUBLISHED",                                │
│   ...                                                    │
│ }                                                        │
└─────────────────────────────────────────────────────────┘
```

---

## Jury Q&A Preparation

### Why Questions - Architecture Decisions

#### **Q1: Why use a Service layer between Controller and Repository?**

**Answer:**
We implement the Service layer to provide several critical benefits:

1. **Separation of Concerns**: 
   - Controllers handle HTTP (parsing requests, formatting responses)
   - Services handle business logic (validation, calculations, workflows)
   - Repositories handle data access (SQL queries)

2. **Reusability**: 
   - Services can be called from multiple controllers
   - Can be invoked by scheduled tasks, event listeners
   - Example: `EmailService.sendEmail()` called from 5+ places

3. **Testability**: 
   - Easy to unit test services by mocking repositories
   - Can test business logic without database
   - Much faster test execution

4. **Transaction Management**: 
   - `@Transactional` at service level ensures ACID properties
   - All-or-nothing operations (if one step fails, all roll back)
   - Example: confirming inscription must simultaneously:
     - Update inscription status
     - Decrement available slots
     - OR rollback both

5. **Security**: 
   - Implement authorization logic in service
   - Prevent unauthorized business operations
   - Log all business-critical actions

```java
// Example: Without service, logic in controller (bad)
@PostMapping("/confirmer/{id}")
public void confirm(@PathVariable Long id) {
    Inscription i = repo.findById(id).get();
    i.setStatus(CONFIRMED);
    i.getOffre().setPlaces(i.getOffre().getPlaces() - 1);
    repo.save(i);  // ← Not atomic! If save fails, logic is inconsistent
}

// With service (good)
@PostMapping("/confirmer/{id}")
public void confirm(@PathVariable Long id) {
    inscriptionService.confirmer(id);  // ← Handles all logic transactionally
}

@Transactional  // ← All-or-nothing
public void confirmer(Long id) {
    Inscription i = inscriptionRepository.findById(id).get();
    i.setStatus(CONFIRMED);
    i.getOffre().setPlaces(i.getOffre().getPlaces() - 1);
    inscriptionRepository.save(i);  // ← Commits atomically
}
```

#### **Q2: Why use Event-Driven Architecture instead of direct method calls?**

**Answer:**
Event-driven architecture provides:

1. **Loose Coupling**: 
   - Service doesn't need to know about email/notification implementation
   - Can swap email providers (SMTP ↔ SendGrid) without touching service
   - Example: If email fails, offer creation succeeds anyway

2. **Asynchronous Processing**: 
   ```
   Without events:
   POST /api/offres/creer
   ├─ Save offer (fast, 10ms)
   ├─ Send 1000 emails (slow, 30s) ← Blocks request!
   └─ Return response (total 30s)
   
   With events:
   POST /api/offres/creer
   ├─ Save offer (10ms)
   ├─ Publish event (1ms)
   └─ Return response immediately (11ms) ← User sees result immediately!
   
   Meanwhile, emails sent in background:
   ├─ OffreEventListener receives event (async, thread pool)
   ├─ Sends 1000 emails (no impact on user request)
   └─ Handles failures gracefully
   ```

3. **Scalability**: 
   - Can add new listeners without changing existing code
   - New listener for Slack notification? Just add `SlackNotificationListener`
   - New requirement for push notifications? Add `PushNotificationListener`

4. **Resilience**: 
   - Email service crashes? Offer creation still succeeds
   - Failed emails can be retried independently
   - Can publish `EmailFailedEvent` for manual retry

5. **Observability**: 
   - Easy to log all events in system
   - Can create audit trail
   - Can implement event sourcing later

#### **Q3: Why Entity Inheritance (JOINED strategy) instead of single table?**

**Answer:**

```
Strategy Comparison:

1. SINGLE_TABLE: All users in one table
   Table: user
   ├─ id, email, password, role
   ├─ admin_special_field_1
   ├─ admin_special_field_2
   ├─ bureau_special_field_1
   ├─ bureau_special_field_2
   └─ adherent_special_field_1
   
   ❌ Sparse data: Admin rows have NULL bureau fields
   ❌ No constraints: Can't require bureau-specific fields
   ✅ Fast queries: Single table join

2. JOINED: Separate tables, linked by ID (our choice)
   Table: user (shared fields)
   ├─ id, email, password, role (not null)
   
   Table: admin
   ├─ id (FK to user)
   └─ admin_special_field (not null)
   
   Table: bureau_member
   ├─ id (FK to user)
   └─ bureau_special_field (not null)
   
   ✅ Data integrity: Not-null constraints per type
   ✅ Type safety: Each role has required fields
   ✅ Polymorphic queries: Query all users
   ⚠️ Slightly slower: Requires joins

3. TABLE_PER_CLASS: Separate complete tables
   Table: admin
   ├─ id, email, password, role, admin_field
   
   Table: bureau_member
   ├─ id, email, password, role, bureau_field
   
   ❌ Data duplication: Email/password duplicated
   ❌ ID conflicts: Same ID in multiple tables
   ❌ Polymorphic queries: UNION required
```

We chose JOINED because:
- **Type Safety**: Each role type enforces its own constraints
- **Polymorphism**: Can query all users: `userRepository.findAll()`
- **Data Integrity**: Shared fields only defined once
- **Relationship Integrity**: Can link to User from Offre without knowing subtype

```java
// Polymorphic query with JOINED
List<User> allUsers = userRepository.findAll();
for (User user : allUsers) {  // ← Can be Admin, Bureau, or Adherent
    if (user instanceof Admin) {
        Admin admin = (Admin) user;
        // Access admin-specific methods
    }
}
```

#### **Q4: Why Mapper pattern instead of MapStruct library?**

**Answer:**

Looking at `OffreMapper.java`, we use **manual mapping** instead of auto-mappers like MapStruct because:

1. **Complex Transformations**: 
   ```java
   // Images need Base64 encoding for JSON response
   if (offre.getImage() != null) {
       res.setImageBase64(Base64.getEncoder().encodeToString(offre.getImage()));
   }
   ```
   MapStruct can't do this without custom @Named methods.

2. **Conditional Logic**: 
   ```java
   // Only set image details if image exists
   if (offre.getImage() != null) {
       res.setImageType(offre.getImageType());
       res.setImageNom(offre.getImageNom());
   }
   ```

3. **Nested Object Extraction**: 
   ```java
   // Extract pole name and ID from nested object
   if (offre.getPole() != null) {
       res.setPoleId(offre.getPole().getId());
       res.setPoleNom(offre.getPole().getNom());
   }
   ```

4. **Error Handling**: 
   ```java
   try {
       // Handle potential lazy loading exceptions
       if (offre.getImagesSupplementaires() != null) {
           res.setImagesSupplementaires(...);
       }
   } catch (org.hibernate.LazyInitializationException ignored) {
       // Image not loaded, skip it
   }
   ```

**However**, we could use MapStruct for **simpler entities** with custom @Mapping annotations:
```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    
    @Mapping(source = "id", target = "userId")
    @Mapping(source = "nom", target = "lastName")
    UserResponse toResponse(User user);
}
```

---

#### **Q5: Why JWT (JSON Web Tokens) instead of session-based authentication?**

**Answer:**

```
Session-Based vs JWT:

SESSION-BASED (Traditional)
Client                          Server
│                               │
├─ Login with credentials       │
├──────────────────────────────→│
│                               ├─ Verify credentials
│                               ├─ Create session (stored in server memory/Redis)
│                               ├─ Send session ID in cookie
│←──────────────────────────────┤
│ (Auto-sent with every request)│
│                               │
├─ GET /api/offres + sessionId  │
├──────────────────────────────→│
│                               ├─ Lookup session in memory
│←──────────────────────────────┤ (Database query each request!)
│ Return offers                 │

❌ Stateful: Server must remember all sessions
❌ Not scalable: Multiple servers need shared session store (Redis)
❌ Slower: Database lookup on every request
✅ Secure: Session data stays on server

JWT-BASED (Modern, Stateless) ← Our choice
Client                          Server
│                               │
├─ Login with credentials       │
├──────────────────────────────→│
│                               ├─ Verify credentials
│                               ├─ Create JWT (encode user info + sign)
│                               ├─ Send JWT
│←──────────────────────────────┤
│                               │
├─ GET /api/offres + JWT        │
├──────────────────────────────→│
│                               ├─ Verify JWT signature
│                               ├─ Decode user info
│←──────────────────────────────┤ (No database lookup!)
│ Return offers                 │

✅ Stateless: Server doesn't store tokens
✅ Scalable: Any server can verify token
✅ Faster: No session lookup (cryptographic verification)
✅ Mobile/API friendly: Works with REST/mobile apps
⚠️ Can't revoke instantly: Token valid until expiry
```

We use JWT because:

1. **Scalability**: 
   ```
   With sessions: Need Redis/shared store
   With JWT: Any server instance can verify
   ```

2. **Mobile-Friendly**: 
   ```
   Sessions use cookies (don't work well on mobile)
   JWT can be stored in localStorage/app storage
   ```

3. **Microservices-Ready**: 
   ```
   Sessions: Auth server must be shared
   JWT: Each service verifies token independently
   ```

4. **Simpler Infrastructure**: 
   ```
   No need for: Redis, session store, session clustering
   Only need: Private key for signing (can be file or env var)
   ```

**JWT Structure:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQGV4YW1wbGUuY29tIiwicm9sZSI6IkFETUlOIiwiaWF0IjoxNjEzMjI0MDAwfQ.signature
│                                      │                                                           │
├─ Header (algorithm + type)            ├─ Payload (claims: user info, role, expiry)             └─ Signature (verify integrity)
└─ Base64 encoded                       └─ Base64 encoded                                         └─ HMAC-SHA256 signed

Decoded:
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "sub": "user@example.com",           ← Email from JWT claim
  "role": "ADMIN",                     ← Role from JWT claim
  "iat": 1613224000,                   ← Issued at
  "exp": 1613310400                    ← Expires at
}
```

---

#### **Q6: Why use @Transactional at service level?**

**Answer:**

```java
@Transactional  // ← Database transaction boundary
public void confirmerInscription(Long id) {
    // 1. Fetch inscription
    Inscription inscription = inscriptionRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Not found"));
    
    // 2. Update inscription
    inscription.setStatut(InscriptionStatus.CONFIRMED);
    inscription.setConfirmedAt(LocalDateTime.now());
    
    // 3. Update offer capacity
    Offre offre = inscription.getOffre();
    offre.setPlacesRestantes(offre.getPlacesRestantes() - 1);
    
    // 4. Save both (both save operations are part of same transaction)
    inscriptionRepository.save(inscription);
    offreRepository.save(offre);
    
    // 5. Publish event
    applicationEventPublisher.publishEvent(
        new InscriptionStatusChangedEvent(inscription, CONFIRMED)
    );
    
    // ← At this point, transaction commits atomically
    // ← If any exception: ALL changes rollback (both inscription and offre)
}
```

**Why This Matters:**

```
Scenario: User confirms 100 inscriptions for offer with 50 places

❌ WITHOUT @Transactional:
├─ Iteration 1: Save inscription #1 ✓ (Commit #1 - permanent!)
├─ Iteration 2: Save inscription #2 ✓ (Commit #2 - permanent!)
├─ ...
├─ Iteration 50: Save inscription #50 ✓ (Commit #50 - permanent!)
├─ Iteration 51: Check places (0 remaining)
└─ Throw exception and stop ✗
   Problem: 50 inscriptions confirmed but offer capacity exceeded!

✅ WITH @Transactional:
├─ Iteration 1: Save inscription #1 (NOT committed yet)
├─ Iteration 2: Save inscription #2 (NOT committed yet)
├─ ...
├─ Iteration 50: Save inscription #50 (NOT committed yet)
├─ Iteration 51: Check places (0 remaining)
├─ Throw exception
└─ ROLLBACK ALL (none of the 50 are saved) ✓
   Consistent state: Offer capacity respected!
```

**Transaction Properties (ACID):**

```
A - Atomicity: All-or-nothing
    All operations in transaction succeed, or all rollback
    
C - Consistency: Data consistency rules maintained
    No orphaned records, foreign key integrity preserved
    
I - Isolation: Changes not visible until commit
    Two concurrent transactions don't interfere
    
D - Durability: Committed data persists
    Even if server crashes, committed data recovers
```

---

### Why Questions - Technical Choices

#### **Q7: Why use RxJS Observables instead of Promises in Angular?**

**Answer:**

```typescript
// Promises: Single value, not cancellable
const promise = fetch('/api/offres').then(res => res.json());
// Can't cancel once started

// Observables: Multiple values, cancellable
const observable = this.http.get('/api/offres');

// Benefits:
1. Cancellable
const subscription = this.http.get('/api/offres').subscribe(...);
subscription.unsubscribe();  // ← Cancel request

2. Retryable
this.http.get('/api/offres').pipe(
  retry(3),  // ← Automatic retry on failure
  catchError(...)
).subscribe(...);

3. Chainable operators
this.http.get('/api/offres').pipe(
  map(offres => offres.filter(o => o.active)),
  filter(offres => offres.length > 0),
  tap(offres => console.log('Found:', offres.length)),
  debounceTime(300)  // ← Wait 300ms before emitting
).subscribe(...);

4. Lazy evaluation
const offres$ = this.http.get('/api/offres');
// Not executed yet!
offres$.subscribe(...);  // ← Executed when subscribed
```

Example Use Case:
```typescript
// Search as user types (debounced)
searchControl = new FormControl('');

constructor(private offreService: OffreService) {}

ngOnInit() {
  this.searchControl.valueChanges
    .pipe(
      debounceTime(300),        // ← Wait 300ms after user stops typing
      distinctUntilChanged(),    // ← Only if value changed
      switchMap(query =>         // ← Cancel previous request
        this.offreService.search(query)
      )
    )
    .subscribe(results => {
      this.searchResults.set(results);
    });
}
```

#### **Q8: Why Angular Signals instead of traditional change detection?**

**Answer:**

```typescript
// Traditional Angular (slower)
export class ComponentA {
  count = 0;
  
  increment() {
    this.count++;
  }
}

// Component needs:
// - ChangeDetectorRef injection
// - Manual markForCheck() calls in some cases
// - Zone.js patches all async operations
// - Every component checked on any change (slow)

// Angular Signals (faster, modern)
export class ComponentA {
  count = signal(0);
  
  increment() {
    this.count.update(v => v + 1);
  }
}

// Benefits:
// - Fine-grained reactivity: Only affected components update
// - No Zone.js needed: Better performance
// - Simpler code: No ChangeDetectorRef injection
// - Explicit dependencies: computed() shows what depends on what

// Computed example
count = signal(0);
doubleCount = computed(() => this.count() * 2);
quadrupleCount = computed(() => this.doubleCount() * 2);

// Change detection tree:
// Traditional: entire app checked
// Signals: count → doubleCount → quadrupleCount (3 checks)
```

Performance Impact:
```
Traditional Change Detection:
- OnPush strategy: Component + all children checked
- Huge form: 100+ fields, entire form re-renders on any change

Signals:
- User types in field #1
- Only field #1 and computed fields depending on it re-render
- Fields #2-100 completely unchanged (not even checked)
```

---

#### **Q9: Why Lazy Loading for feature modules?**

**Answer:**

```typescript
// Without lazy loading
{
  path: 'admin',
  component: AdminDashboardComponent  // ← Bundled with main app
}

// User downloads JavaScript:
// ├─ Angular core (200KB)
// ├─ Auth module (50KB)
// ├─ Admin module (500KB) ← Downloaded even if user is not admin!
// └─ Adherent module (200KB) ← Never used!
// Total: 950KB (but user only needs 450KB for their role)

// With lazy loading
{
  path: 'admin',
  canActivate: [adminGuard],
  loadComponent: () => 
    import('./admin/layout').then(m => m.AdminLayoutComponent)
}

// User downloads:
// Initial: 450KB (main + auth + adherent)
// On admin access: +500KB (admin module loaded on-demand)
// Non-admin users: Never download admin module!

// Results:
// - 47% faster initial load (950KB → 450KB)
// - Better mobile experience
// - Smaller JavaScript bundle
```

---

### Implementation Questions

#### **Q10: How does JWT token validation work on every request?**

**Answer:**

```
Request Flow:

1. Browser sends request with JWT
   GET /api/offres
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

2. JwtAuthFilter intercepts request
   @Component
   public class JwtAuthFilter extends OncePerRequestFilter {
       
       @Override
       protected void doFilterInternal(
           HttpServletRequest request,
           HttpServletResponse response,
           FilterChain filterChain) throws ServletException, IOException {
           
           try {
               // Extract Bearer token
               String header = request.getHeader("Authorization");
               String token = header.substring(7);  // Remove "Bearer "
               
               // Verify signature (uses secret key)
               if (jwtUtils.isValidToken(token)) {
                   // Decode token (no database lookup!)
                   String email = jwtUtils.getEmailFromToken(token);
                   
                   // Load user from database (cache in production)
                   User user = userRepository.findByEmail(email)
                       .orElseThrow(UnauthorizedException::new);
                   
                   // Create authentication with user's roles
                   UserPrincipal principal = new UserPrincipal(user);
                   Authentication auth = new UsernamePasswordAuthenticationToken(
                       principal,
                       null,
                       principal.getAuthorities()  // [ROLE_ADMIN]
                   );
                   
                   // Store in SecurityContext (available in @AuthenticationPrincipal)
                   SecurityContextHolder.getContext().setAuthentication(auth);
               }
               
               filterChain.doFilter(request, response);
           } catch (JwtException | UnauthorizedException e) {
               response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               response.getWriter().write("Invalid token");
           }
       }
   }

3. Controller/Service can access authenticated user
   @PostMapping("/offres")
   @PreAuthorize("hasRole('ADMIN')")  // ← Checks authentication
   public ResponseEntity<OffreResponse> create(
       @RequestBody OffreRequest req,
       @AuthenticationPrincipal UserDetails userDetails  // ← Current user
   ) {
       String email = userDetails.getUsername();  // ← Extracted from JWT
       // ... create offer
   }

4. Response sent with appropriate status
   ✓ 200 OK - If authorized and operation succeeded
   ✗ 401 Unauthorized - If token invalid/missing
   ✗ 403 Forbidden - If user lacks required role
   ✗ 404 Not Found - If resource not found
```

**Security Advantages:**

```
1. Signature verification prevents tampering
   User can't modify JWT (would invalidate signature)
   
   Original: 
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiQURN...
   
   If user changes role in payload:
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiQURN...
   └─ Signature mismatch! Token rejected.

2. Token expiry prevents indefinite access
   {
     "email": "user@example.com",
     "exp": 1613310400,  // Expires in 1 hour
   }
   
   After 1 hour, token rejected even if valid signature

3. Backend retains verification power
   Even if token looks valid, backend verifies with database
   If user disabled in DB → 403 Forbidden
```

---

#### **Q11: How does @EventListener handle asynchronous event processing?**

**Answer:**

```java
// Event published in main transaction
@Service
@RequiredArgsConstructor
public class OffreService {
    
    @Transactional
    public OffreResponse creer(OffreRequest req) {
        Offre offre = offreRepository.save(...);  // 1. Saved to DB
        
        // 2. Publish event (non-blocking, immediate return)
        applicationEventPublisher.publishEvent(
            new OffreCreatedEvent(offre)  // ← Event created
        );  // ← Returns immediately!
        
        return offreMapper.toResponse(offre);  // 3. Return to client immediately
    }
}
// ← Transaction commits here
// ← HTTP response returned to client
// ← Meanwhile, event listeners still processing...

// Event listener (async, separate thread)
@Component
@RequiredArgsConstructor
public class OffreEventListener {
    
    @Async                      // ← Runs in ThreadPoolTaskExecutor
    @EventListener              // ← Triggered by OffreCreatedEvent
    public void onOffreCreated(OffreCreatedEvent event) {
        Offre offre = event.offre();
        
        // This runs AFTER main transaction commits
        // In a separate thread (from ThreadPool)
        // Non-blocking to user request
        
        try {
            // 1. Find all active users
            List<User> users = userRepository.findByActifTrue();
            
            // 2. Send emails (slow operation)
            for (User user : users) {
                emailService.sendOffreNotification(user.getEmail(), offre);
            }
            
            // 3. Update cache
            cacheService.invalidateOffreCache();
            
            log.info("Processed OffreCreatedEvent for offre: {}", offre.getId());
        } catch (Exception e) {
            log.error("Error processing OffreCreatedEvent", e);
            // Error doesn't affect main transaction (already committed)
            // Could publish EmailFailedEvent for retry mechanism
        }
    }
}

// Configuration for async processing
@Configuration
@EnableAsync  // ← Enable @Async annotation
public class AsyncConfig {
    
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);           // ← Min threads
        executor.setMaxPoolSize(20);           // ← Max threads
        executor.setQueueCapacity(100);        // ← Queue size
        executor.setThreadNamePrefix("async-"); // ← Thread naming
        executor.initialize();
        return executor;
    }
}

// Timeline of events:
User makes request
│
├─ T0: POST /api/offres/creer
│
├─ T10ms: OffreService.creer() starts
│   ├─ Save to DB (5ms)
│   ├─ Publish event (1ms)
│   └─ Return OffreResponse (1ms)
│
├─ T17ms: HTTP response returned to client ← User sees result!
│
├─ T18ms: OffreEventListener receives event
│   ├─ Thread Pool picks up task
│   ├─ Send 1000 emails (30s)
│   ├─ Update cache (2s)
│   └─ Complete
│
└─ T50s: All background work done
   (But client already has response after 17ms!)
```

---

#### **Q12: How does Entity inheritance (JOINED) work in practice?**

**Answer:**

```java
// Database schema
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    dtype VARCHAR(31),  // ← Discriminator (ADMIN, BUREAU, ADHERENT)
    email VARCHAR(255) UNIQUE NOT NULL,
    mot_de_passe VARCHAR(255) NOT NULL,
    nom VARCHAR(255),
    prenom VARCHAR(255),
    role ENUM('ADMIN','BUREAU','ADHERENT')
);

CREATE TABLE admin (
    id BIGINT PRIMARY KEY,
    FOREIGN KEY (id) REFERENCES user(id)
    // ← No additional columns needed
);

CREATE TABLE membre_bureau (
    id BIGINT PRIMARY KEY,
    pole_id BIGINT,
    FOREIGN KEY (id) REFERENCES user(id),
    FOREIGN KEY (pole_id) REFERENCES pole(id)
);

CREATE TABLE adherent (
    id BIGINT PRIMARY KEY,
    numero_adhesion VARCHAR(255),
    date_adhesion DATE,
    FOREIGN KEY (id) REFERENCES user(id)
);

// JPA Entity Hierarchy
@Entity
@Table(name = "user")
@Inheritance(strategy = InheritanceType.JOINED)  // ← JOINED strategy
@DiscriminatorColumn(name = "dtype")             // ← Discriminator column
@Getter @Setter
public abstract class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String motDePasse;
    
    @Enumerated(EnumType.STRING)
    private Role role;
}

@Entity
@Getter @Setter
public class Admin extends User {
    // No additional fields needed
    // All inherited from User
}

@Entity
@Getter @Setter
public class MembreBureau extends User {
    
    @ManyToOne
    @JoinColumn(name = "pole_id")
    private Pole pole;  // ← Bureau-specific field
}

@Entity
@Getter @Setter
public class Adherent extends User {
    
    @Column(unique = true)
    private String numeroAdhesion;  // ← Adherent-specific field
    
    private LocalDate dateAdhesion;
}

// Usage
// Create entities
Admin admin = new Admin();
admin.setEmail("admin@example.com");
admin.setRole(Role.ADMIN);
userRepository.save(admin);  // ← Saves to user table + admin table

MembreBureau bureau = new MembreBureau();
bureau.setEmail("bureau@example.com");
bureau.setRole(Role.MEMBRE_BUREAU);
bureau.setPole(poleRepository.findById(1L).get());
userRepository.save(bureau);  // ← Saves to user table + membre_bureau table

// Polymorphic queries
List<User> allUsers = userRepository.findAll();
// ← Returns mix of Admin, MembreBureau, Adherent
// ← SQL: LEFT JOIN admin a ON u.id = a.id
//        LEFT JOIN membre_bureau mb ON u.id = mb.id
//        LEFT JOIN adherent ad ON u.id = ad.id

for (User user : allUsers) {
    if (user instanceof Admin) {
        Admin admin = (Admin) user;
        // Access admin-specific methods
    } else if (user instanceof MembreBureau) {
        MembreBureau bureau = (MembreBureau) user;
        Pole pole = bureau.getPole();  // ← Bureau-specific field
    }
}

// Find specific type
List<Admin> admins = adminRepository.findAll();
List<MembreBureau> bureauMembers = membreBureauRepository.findAll();
```

**Database Queries Generated:**

```sql
-- Inserting Admin
INSERT INTO user (dtype, email, mot_de_passe, role) 
VALUES ('ADMIN', 'admin@example.com', '$2a$10$...', 'ADMIN');

INSERT INTO admin (id) 
VALUES (LAST_INSERT_ID());

-- Inserting MembreBureau
INSERT INTO user (dtype, email, mot_de_passe, role) 
VALUES ('MembreBureau', 'bureau@example.com', '$2a$10$...', 'MEMBRE_BUREAU');

INSERT INTO membre_bureau (id, pole_id) 
VALUES (LAST_INSERT_ID(), 1);

-- Fetching all users (polymorphic)
SELECT u.*, a.*, mb.*, ad.* 
FROM user u
LEFT JOIN admin a ON u.id = a.id
LEFT JOIN membre_bureau mb ON u.id = mb.id
LEFT JOIN adherent ad ON u.id = ad.id;

-- Fetching admins only
SELECT u.* FROM user u 
WHERE u.dtype = 'ADMIN';
```

---

### Q&A Summary Table

| Question | Key Takeaway |
|----------|--------------|
| Q1: Why Service layer? | Separation of concerns, testability, transaction management |
| Q2: Why Event-Driven? | Loose coupling, async processing, scalability |
| Q3: Why JOINED inheritance? | Type safety, polymorphism, data integrity |
| Q4: Why manual mappers? | Complex transformations, business logic, error handling |
| Q5: Why JWT? | Stateless, scalable, mobile-friendly |
| Q6: Why @Transactional? | ACID properties, atomicity, consistency |
| Q7: Why Observables? | Cancellable, retryable, chainable, lazy evaluation |
| Q8: Why Signals? | Fine-grained reactivity, better performance |
| Q9: Why Lazy Loading? | Smaller initial bundle, faster initial load |
| Q10: JWT Validation? | Signature verification, token expiry, database consistency |
| Q11: Async Events? | Non-blocking requests, scalable architecture |
| Q12: Entity Inheritance? | Database normalization, type safety, polymorphic queries |

---

## Best Practices Implemented

### 1. **Input Validation**
- File size limits (1MB per image)
- Email regex validation
- Required field validation
- Business rule validation (capacity checks)

### 2. **Security**
- BCrypt password hashing
- JWT token signing with HMAC
- Role-based access control (@PreAuthorize)
- Method-level security

### 3. **Error Handling**
- Custom exception classes
- Meaningful error messages
- Proper HTTP status codes
- Error logging and monitoring

### 4. **Performance**
- Read-only transactions (@Transactional(readOnly=true))
- Lazy loading strategy
- Database indexing (email, created_at)
- Caching (can be added for frequently accessed data)

### 5. **Maintainability**
- Clear package structure
- Interface-based design
- Dependency injection
- Comprehensive logging

### 6. **Testing**
- Unit tests for services
- Integration tests for controllers
- Test database (H2) configuration
- Mocking external dependencies

---

## Conclusion

The **AMICALE-STAR** platform demonstrates modern software architecture principles:

- **Clean Code**: Clear separation of concerns
- **Scalability**: Event-driven, async processing
- **Security**: JWT, encryption, role-based access
- **Maintainability**: Interfaces, dependency injection, testable design
- **User Experience**: Stateless API, fast response times
- **Type Safety**: Java generics, validation, entity inheritance

The architecture is production-ready and follows industry best practices used by major tech companies.

---

