![License](https://img.shields.io/github/license/ADORSYS-GIS/taler-obg)
![Issues](https://img.shields.io/github/issues/ADORSYS-GIS/taler-obg)

# taler-obg

**taler-obg** is an open source extension of the OBG project. It extends OBG for use in the NGI Taler project, improving user experience, and adding new adaptors for European banks. With a focused scope banks' PIS integration to support automatic top-ups in the Taler Wallet.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Repository Structure](#repository-structure)
- [Project Organization](#project-organization)
  - [Daily Meetings](#daily-meetings)
  - [Milestone Progress & Sprint Planning](#milestone-progress--sprint-planning)
  - [Reviews and Retrospectives](#reviews-and-retrospectives)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [License](#license)
- [Contact & Communication](#contact--communication)

---

## Overview

**taler-obg** extends the open source OBG project by integrating:
- Enhanced deployment automation with updated Ansible scripts.
- Improved backend components (e.g., updated Java libraries, enhanced APIs).
- Adaptors to connect with European banks via open banking interfaces.
- A streamlined user experience for wallet top-ups and account verification using PIS.

The project is driven by community collaboration and aims to provide a production-ready solution that can be further extended by both operators and independent contributors.

---

## Key Features

- **Enhanced API and Workflow Support:**  
  Deep integration with the OBG FinTech API, supporting multiple Strong Customer Authentication (SCA) approaches and the XS2A interface for payment status queries.

- **Modular Architecture:**  
  A flexible design that allows for additional adaptors and functionalities to be added as needed.

- **Automated Deployment:**  
  Updated Ansible scripts that simplify installation and deployment processes.

- **User Experience Improvements:**  
  Focus on a seamless top-up process and enhanced account verification within merchant apps.

---

## Repository Structure

The repository is organized as follows:

```
taler-obg/
├── README.md                # This file
├── docs/                    # Documentation files
│   ├── architecture.md      # Detailed architectural overview
│   ├── sprint-planning.md   # Sprint plans and backlog details
│   ├── meetings/            # Meeting logs and summaries
│   │   ├── daily.md         # Daily meeting notes
│   │   ├── reviews.md       # Sprint reviews and demos
│   │   ├── retrospectives.md# Retrospective summaries
│   │   └── milestone-progress.md # Milestone progress status
│   ├── contribution.md      # Guidelines for contributing
│   ├── installation.md      # Installation and setup instructions
│   └── user-guide.md        # Usage instructions and examples
├── src/                     # Source code for the project
├── tests/                   # Test suites and scripts
├── ansible/                 # Ansible scripts for deployment
└── LICENSE                  # License file
```

This structure is designed to keep both technical documentation and project management artifacts (e.g., meeting notes, sprint planning) within the repository—making it easy for contributors and users to access everything in one place.

---

## Project Organization

To keep the project transparent and well-managed, we document our organizational processes here.

### Daily Meetings

- **Format:**  
  Short stand-ups to track progress, discuss blockers, and align on daily tasks.
- **Documentation:**  
  Daily meeting notes are stored in `docs/meetings/daily.md`. Contributors are encouraged to update this file with summaries after each meeting.

### Milestone Progress & Sprint Planning

- **Milestones:**  
  The project follows a milestone-based approach. Each milestone focuses on distinct areas such as backend updates, frontend improvements, and integration of bank adaptors.
- **Sprint Planning:**  
  Sprint planning sessions (including task assignments, prioritization, and review of sprint goals) are documented in `docs/sprint-planning.md`.
- **Progress Tracking:**  
  A dedicated status page is maintained in `docs/meetings/milestone-progress.md` to track ongoing progress and upcoming tasks.

### Reviews and Retrospectives

- **Sprint Reviews:**  
  Outcomes from sprint reviews and demos are documented in `docs/meetings/reviews.md`.
- **Retrospectives:**  
  Lessons learned and areas for improvement are captured in `docs/meetings/retrospectives.md`.

### Project Management

- **Issue Tracking & Project Board:**  
  Our project issues and tasks are managed on GitHub. You can view and contribute to the project board here: [ADORSYS-GIS Project Board](https://github.com/orgs/ADORSYS-GIS/projects/22).
  
---

## Getting Started

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/ADORSYS-GIS/taler-obg.git
   cd taler-obg
   ```

2. **Installation:**
   - Refer to the [Installation Guide](docs/installation.md) for detailed steps to set up your development environment, run tests, and deploy the solution.

3. **Documentation:**
   - For an in-depth overview of the system architecture and component design, see [Architecture](docs/architecture.md).

4. **Testing:**
   - Run the provided test suites located in the `tests/` directory to verify the installation.

---

## Contributing

We welcome contributions from the community! Please check out our [Contribution Guidelines](docs/contribution.md) for instructions on how to get started, coding standards, and submission processes.

---

## License

This project is licensed under the [AGPL v3](LICENSE).

---

## Contact & Communication

For further discussions, issues, or contributions:
- Join our Mattermost channel for real-time updates.
- Open issues or pull requests directly in the repository.
- For additional questions, please contact the project maintainers.

---

*Happy Coding!*
