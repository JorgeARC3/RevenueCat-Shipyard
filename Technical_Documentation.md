# DreamPath: Technical Documentation

## 1. Technology Stack
DreamPath is built on a modern, reactive architecture designed for high availability and offline resilience.

*   **Frontend**: [Flutter SDK](https://flutter.dev/) (Cross-platform Android/iOS).
*   **State Management**: [Riverpod](https://riverpod.dev/) (Unidirectional data flow, type-safety).
*   **Backend as a Service (BaaS)**: [Firebase](https://firebase.google.com/).
    *   **Firestore**: Real-time document storage.
    *   **Auth**: Secure user management (Email/Password & Social).
    *   **Messaging**: Push notifications for daily task reminders.
*   **AI Engine**: [OpenRouter API](https://openrouter.ai/) (Utilizing the **GPT-OSS-120B** model) for structured JSON plan generation.
*   **Monetization**: [RevenueCat SDK](https://www.revenuecat.com/) for subscription management and paywall orchestration.

## 2. System Architecture
The application follows a **"Service-Provider-Pattern"** to ensure clean separation of concerns and high testability.

### Key Layers:
1.  **UI Layer**: Flutter widgets using `ConsumerWidget` for reactive state updates.
2.  **Provider Layer**: Riverpod `Notifier` classes (e.g., `DreamsNotifier`) that handle business logic.
3.  **Service Layer**: Individual services for Firebase, RevenueCat, AI, and Local Storage.
4.  **Data Layer**: JSON-serializable models and offline queue management.

### Offline-First Logic:
To ensure a premium UX, DreamPath implements an **Offline Queue Service**.
*   All operations (logging a win, creating a dream) are saved immediately to `SharedPreferences` and local state.
*   An background worker attempts to sync these operations to Firestore when the internet connection is restored.

## 3. RevenueCat Implementation
RevenueCat acts as the single source of truth for user entitlement state, decoupling monetization logic from the App Store and Play Store complexities.

### Implementation Flow:
1.  **Configuration**: Initialized on app startup via `RevenueCatService`.
2.  **User Identification**: On Firebase Auth success, the user's UID is passed to RevenueCat using `Purchases.logIn(uid)`.
3.  **Offerings**: The app fetches current `Offerings` to display the dynamic Paywall.
4.  **Entitlement Check**: We use a `subscriptionProvider` in Riverpod that listens to `CustomerInfo` updates. 
    *   Feature access (e.g., creating the 4th dream) is gated by checking `customerInfo.entitlements.active['premium']`.

### Why RevenueCat?
*   **Speed of Implementation**: Reduced our monetization setup time by ~70%.
*   **Reliability**: Handles edge cases like receipt validation and trial management automatically.
*   **Marketing Agility**: Allows us to change pricing or offer trials via the RevenueCat Dashboard without a new app release.

## 4. AI Workflow
The AI service uses "Reasoning-Enabled" prompts to ensure the roadmaps are not just generic lists, but logically progressive plans tailored to the user's category (Travel, Business, etc.).

*   **Input**: User Dream + Category + Timeline.
*   **Prompt**: System instructions enforced strict JSON output.
*   **Post-processing**: The `AiService` injects "Gabby Tips" (hardcoded community wisdom) based on the dreamer's niche to add a human, community-centric touch.
