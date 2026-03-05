# WhatsApp AI Booking Assistant for Shortlet Rentals (n8n Automation)

AI-powered WhatsApp front desk assistant built with n8n that automates shortlet booking conversations.

The system handles apartment inquiries, retrieves listing data from Google Sheets, maintains conversation context with Redis, and guides guests through the booking process using an AI agent.

## Stack

- n8n
- Evolution API (WhatsApp)
- OpenAI
- Redis
- Google Sheets

## Features

- WhatsApp conversation automation
- Apartment availability checks
- AI-powered responses
- Booking qualification
- Payment instruction generation

## Workflow

WhatsApp → Evolution API → Message Parser → Redis Memory → AI Agent → Google Sheets → Response
