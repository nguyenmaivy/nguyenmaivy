---
title: "Worklog Week 9"

weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

### Objectives of Week 9:

* Implement user authentication using AWS Cognito.
* Develop chat and favorite features for user interaction.
* Optimize UI for responsive design.

### Tasks Implemented During the Week:

| Category | Detailed Tasks | Deliverables |
|----------|----------------|--------------|
| **Authentication (Auth)** | * **AWS Cognito Setup:** Create User Pool and App Client. <br> * **Frontend:** Integrate **AWS Amplify** into Next.js. <br> * Build UI for **Sign Up / Sign In / Forgot Password**. <br> * **User Roles:** Implement basic authorization (User vs Owner) using Cognito Groups. | Users can **successfully register and log in**, and the system correctly distinguishes user roles. |
| **Chat & Favorite** | * **Backend:** Complete logic in `chatMessage.js` (save/load messages between two users) and `favorite.js` (add/remove favorite rooms). <br> * **Frontend:** Build **Chat Modal/Page** (`ChatModal.js`, `chat/page.js`) for message sending/receiving. <br> * Integrate **Favorite button** on `RoomCard.js`. | Chat and Favorite features work reliably and smoothly. |
| **UI Optimization** | * Ensure all main pages are **fully responsive** (Mobile-first design). | User-friendly interface on all devices. |

### Results of Week 9:

* Authentication system using **AWS Cognito** is fully operational.
* Successfully implemented **real-time chat interaction** between users.
* Users can **save favorite rooms**.
* UI optimized for **mobile, tablet, and desktop** experiences.
* **Overall Outcome:** User interaction features are complete, enhancing overall system usability.
