# Airbnb Distributed System — Full-Stack Cloud-Native Application

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)
![Node.js](https://img.shields.io/badge/Backend-Node.js%2FExpress-brightgreen)
![React](https://img.shields.io/badge/Frontend-React-blue)
![Redux](https://img.shields.io/badge/State-Redux-purple)
![Python](https://img.shields.io/badge/AI%20Agent-FastAPI-orange)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED)
![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5)
![Kafka](https://img.shields.io/badge/Messaging-Kafka-000000)
![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900)

A production-style **Distributed System** mimicking Airbnb's core functionality. This project demonstrates high-performance cloud-native engineering patterns, focusing on scalability, asynchronous communication, and container orchestration.

**Key capabilities:**
- **Full-Stack Implementation**: React Frontend + Node.js/FastAPI Microservices.
- **Event-Driven Architecture**: Asynchronous communication using **Apache Kafka**.
- **Containerization**: Cloud-native deployment strategy with **Docker & Kubernetes**.
- **State Management**: Predictable state flow using **Redux Toolkit**.
- **AI Integration**: Travel planning assistant powered by **LangChain & FastAPI**.
- **Scalability**: Tested for high throughput and fault tolerance using **JMeter**.

---

## System Architecture

The system utilizes a microservices architecture where distinct domains (Traveler, Owner, Property) are decoupled and communicate via HTTP for synchronous operations and Kafka for asynchronous state synchronization (e.g., booking workflows).

```text
┌──────────────────────────────────────────────────────────────────────┐
│                    FRONTEND (React + Redux + Vite)                   │
│                         http://localhost:5173                        │
│                                                                      │
│  Redux Store:                                                        │
│  • authSlice         - User authentication, JWT tokens               │
│  • propertiesSlice   - Property search results, details              │
│  • bookingsSlice     - Booking list, status updates                  │
│  • favoritesSlice    - Favorite properties                           │
│  • dashboardSlice    - Owner dashboard stats                         │
│                                                                      │
│  Technologies: React, Redux Toolkit, TailwindCSS, Axios              │
└──────┬───────────────┬────────────────┬──────────────┬──────────────┘
       │               │                │              │
       │ JWT Auth      │ JWT Auth       │ JWT Auth     │ HTTP
       │               │                │              │
       ↓               ↓                ↓              ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   TRAVELER   │ │    OWNER     │ │   PROPERTY   │ │    AGENT     │
│   SERVICE    │ │   SERVICE    │ │   SERVICE    │ │   SERVICE    │
│  Port: 3001  │ │  Port: 3002  │ │  Port: 3003  │ │  Port: 8000  │
│              │ │              │ │              │ │              │
│  Routes:     │ │  Routes:     │ │  Routes:     │ │  Endpoints:  │
│  /api/auth   │ │  /api/auth   │ │  /api/props  │ │  /agent/plan │
│  /api/favs   │ │  /api/props  │ │  (CRUD)      │ │  /health     │
│  /api/books  │ │  /api/books  │ │              │ │              │
│  (PRODUCER)  │ │  (PRODUCER)  │ │              │ │              │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │                │
       │ Mongoose       │ Mongoose       │ Mongoose       │ Mongoose
       │                │                │                │
       ↓                ↓                ↓                ↓
┌──────────────────────────────────────────────────────────────────────┐
│                        MongoDB (Port: 27017)                         │
│                                                                      │
│  Database: airbnb                                                    │
│                                                                      │
│  Collections:                                                        │
│  • users          - Travelers & Owners (role, email, password)       │
│  • properties     - Property listings (location, price, amenities)   │
│  • bookings       - Bookings (status: PENDING/ACCEPTED/CANCELLED)    │
│  • favorites      - Favorite properties by travelers                 │
│                                                                      │
│  Authentication: admin/adminpassword                                 │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                     KAFKA MESSAGE BROKER                             │
│                  (Port: 9092, UI: 9093)                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │  Topic: booking-requests                                    │     │
│  │  Purpose: New booking creation notifications                │     │
│  │  Producer: Traveler Service                                 │     │
│  │  Consumer: (Optional) Owner Service for notifications       │     │
│  │                                                             │     │
│  │  Message Schema:                                            │     │
│  │  {                                                          │     │
│  │    bookingId, travelerId, propertyId, ownerId,              │     │
│  │    startDate, endDate, totalPrice, status, timestamp        │     │
│  │  }                                                          │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐     │
│  │  Topic: booking-updates                                     │     │
│  │  Purpose: Booking status changes (ACCEPTED/CANCELLED)       │     │
│  │  Producers: Owner Service, Traveler Service                 │     │
│  │  Consumer: Booking Service (status sync)                    │     │
│  │                                                             │     │
│  │  Message Schema:                                            │     │
│  │  {                                                          │     │
│  │    bookingId, status, updatedBy, timestamp                  │     │
│  │  }                                                          │     │
│  └─────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Backed by: Zookeeper (Port: 2181)                                   │
└───────────────┬──────────────────────────────────┬──────────────────┘
                │                                  │
                │ Subscribes                       │ Subscribes
                │ (booking-updates)                │ (booking-updates)
                ↓                                  ↓
      ┌──────────────────┐               ┌──────────────────┐
      │      BOOKING     │               │  (Future)        │
      │      SERVICE     │               │   Email/Alert    │
      │   Port: 3004     │               │   Consumers      │
      │                  │               │                  │
      │  Consumer Group: │               │  e.g., Email     │
      │  booking-status  │               │  Notification    │
      │  -sync-group     │               │  Service         │
      │                  │               └──────────────────┘
      │  Function:       │
      │  Synchronize     │
      │  booking status  │
      │  across services │
      └──────────────────┘
