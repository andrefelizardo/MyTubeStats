# Product Requirements Document (PRD) - YouTube Wrapped SaaS

## Overview

A web application (SaaS) that allows users to view detailed statistics of their YouTube watch history, generating an interactive dashboard similar to "Spotify Wrapped".

## The Problem and the Solution

The official YouTube API does not allow access to a user's watch history due to strict privacy policies.
The solution bypasses this limitation by instructing the user to export their data via Google Takeout (JSON format) and upload this file to our platform for analysis.

## User Journey (Core Loop)

1. The user accesses the SaaS landing page.
2. The user is guided through a step-by-step tutorial on how to generate their YouTube History file via Google Takeout.
3. The user uploads the `.json` or `.zip` file to the platform.
4. The system processes the file, extracts the video IDs, and calculates the total watch time along with other metrics.
5. The user views an interactive dashboard displaying their personalized data.

## Functional Requirements (Features)

- **Upload and Validation:** The system must accept JSON file uploads and validate their structure to ensure it is a valid Google Takeout file.
- **Efficient Processing:** The system must be capable of reading heavy JSON files (ranging from megabytes to gigabytes) without exhausting the server's memory (RAM).
- **Data Enrichment:** The system must use the official YouTube Data API v3 to fetch the actual duration of each video using the IDs extracted from the Takeout file.
- **Dashboard:** Display key metrics such as: total watch time (in hours/days), most-watched channels, and viewing distribution over specific time periods.

## Constraints and Business Rules

- File processing must be done strictly in memory or with immediate disposal; sensitive user data must never be permanently stored without explicit consent.
- The system must gracefully handle the YouTube API rate limits when batch-querying video durations.
