[README.md](https://github.com/user-attachments/files/34651938/README.md)
# GetaJob — Privacy-First Desktop Application Tracker

&gt; **Project Type:** Personal Learning &amp; Engineering Project **Platform:** Windows Desktop (C# / .NET 8 / WPF) **Architecture:** Clean / Layered Architecture

---

## 📋 Executive Summary

**GetaJob** is a privacy-first, local-first desktop application designed to streamline and empower direct job applications for local businesses and nearby opportunities. Built to eliminate recruitment agency gatekeeping and avoid privacy risks associated with third-party job boards, GetaJob enables job seekers to manage their entire job search workflow offline on their local machine.

### Key Functional Highlights

* **Local-First &amp; Privacy-Focused:** All user contacts, notes, status history, and credentials reside strictly on the user's local machine in a local database—eliminating server-side data collection and third-party privacy risks.
* **Streamlined Contact &amp; Application Management:** Tracks business locations, contact details, application states, and custom communication workflows.
* **Direct Email Integration:** Connects securely with the user’s mail accounts (IMAP/SMTP or OAuth) to send and receive outreach directly without exposing middleware servers.
* **Location &amp; Visual Mapping:** Integrates map views to visually organize nearby businesses and target neighborhoods for localized outreach.

---

## 🛠️ Technical Showcase

GetaJob was engineered as a production-grade desktop application using modern C# and .NET standards, demonstrating clean software design principles, decoupled modularity, and native packaging.

### Tech Stack &amp; Frameworks

| Category                   | Technology                            | Usage / Purpose                                                   |
| -------------------------- | ------------------------------------- | ----------------------------------------------------------------- |
| **Language &amp; Runtime**     | C# / .NET 8                           | Core application logic and system API access                      |
| **User Interface**         | WPF (Windows Presentation Foundation) | Native Windows desktop graphical interface                        |
| **Database &amp; Persistence** | SQLite + Dapper                       | Lightweight local database storage and micro-ORM data access      |
| **Email Processing**       | MailKit &amp; MimeKit                     | Robust email MIME creation, parsing, and mail protocol management |
| **Mapping &amp; GIS**          | Mapsui                                | Native map visualization for business geographic coordinates      |
| **Testing &amp; QA**           | xUnit + Coverlet                      | Unit testing, integration test suites, and code coverage metrics  |
| **Packaging &amp; Installer**  | WiX Toolset v4                        | Building native Windows MSI installer packages (`GetaJob.Setup`)  |

### Architecture &amp; Design Patterns

The repository is structured around **Clean / Layered Architecture** to enforce separation of concerns, testability, and maintainability:

* **Domain Layer (** **GetaJob.Domain** **):** Core business entities, contact status rules, and domain models free of external framework dependencies.
* **Application Layer (** **GetaJob.Application** **):** Use case orchestrators, application services, and workflow execution contracts.
* **Infrastructure Layer (** **GetaJob.Infrastructure** **):** Low-level implementations including SQLite data persistence, file-backed storage, and email transport logic.
* **Bootstrap Layer (** **GetaJob.Bootstrap** **):** First-run database initialization, configuration loading, and dependency injection composition.
* **Presentation Layer (** **GetaJob.Desktop** **):** WPF user interface views, data binding, and desktop user flows.

### Engineering &amp; Learning Takeaways

* **Local-First Security:** Practiced secure local credential storage and designed build modes separating internal development/QA dummy workflows from production-only TLS connections.
* **Desktop Application Lifecycle:** Gained experience in full-lifecycle software delivery—from SDK pinning and central package management (`Directory.Packages.props`) to Automated PowerShell publishing scripts (`build-desktop-installer.ps1`) and native MSI installer creation.
* **Decoupled Integration:** Implemented service interfaces to decouple UI components from network/database interactions, simplifying automated unit and integration testing.
