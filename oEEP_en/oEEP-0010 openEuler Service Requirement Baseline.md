---
Title:          openEuler Service Requirement Baseline
Type:           Document
Abstract:       openEuler service capability description and evaluation
Author:         Deng Huilong (Talent-and-Service SIG)
Status:         Initial
No:             oEEP-0010 
Create_time:    2023-08-02
Update_time:    2023-08-02
---

## Motivation/Problem

By referencing the best practices of service systems from leading companies in the industry and considering the characteristics of open source infrastructure software, this document aims to define the service capability requirements for openEuler. These requirements will serve as a reference for community partners to learn from and improve their overall service capabilities.

## Solution Description

### Scope

This document serves as a reference for openEuler community open source edition service capabilities. It is not mandatory. For commercial service contracts, the agreement between the two parties shall be used.
The delivery and usage of an operating system (OS) involves several common service roles:

1. Hardware manufacturer
2. Operating system vendor (OSV)
3. Independent software vendor (ISV)
4. Solution integrator
5. openEuler service provider (oESP)

This baseline mainly focuses on the service scope of oESPs.

### Content

From the user's perspective, this document outlines the service support required from third-party organizations or individuals during the use of openEuler. The relevant support service content is as follows:
|   Capability Category |    Subcategory   | Description |
| ---------| ------------ | ------------ |
| Basic services | Installation and deployment<br>Migration and upgrade<br>Diagnosis and analysis<br>Patch update | Basic usage content and requirements, enabling users to functionally utilize openEuler |
| Advanced services | Performance tuning<br>Security inspection and hardening<br>Health check<br>Dedicated support | Advanced capabilities for openEuler usage, enhancing stability, security, and reducing usage costs  |
| Consulting services | Solution planning<br>Technical consulting | Assisting users in understanding functionalities and designing usage solutions |
| Training and certification services | Training and enablement<br>Evaluation and certification | Providing openEuler usage training for organizational users, including subsequent examinations and certifications|
| Version customization services | Version customization | Supporting users in secondary development based on the original openEuler OS, including software package tailoring, program presetting, and package building|
| Multi-channel support services| Multi-channel support|Providing diverse support channels to meet different service demands, including but not limited to email, WeChat, phone calls, and on-site support|
| Multi-language support services| Multi-language support|Offering services in multiple languages to facilitate consultations for users from various regions|
| Service Records| Service record content<br>Service record retention|Recording service support process information for subsequent analysis and queries|
| Service condition limitations| Service denial content<br>Service denial handling procedures| Listing service limitations due to objective conditions and corresponding solutions|

For each service subcategory, specific requirements for "content requirements" and "service results" will be described.

## Discussion and Review

- After multiple rounds of online and offline reviews with partners within the Talent-and-Service SIG, a consensus has been reached within the SIG.
- The baseline has passed the review at the Technical Committee meeting. This baseline can help community partners improve their service capabilities.

## Implementation Plan

The plan is to release the baseline in the Talent-and-Service SIG repository and update it regularly based on community feedback.
