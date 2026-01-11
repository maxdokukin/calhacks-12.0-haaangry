# Haaangry

## If Tik-Tok, Tinder, and Doordash had a baby

## Demo
[![Watch the Demo](https://img.youtube.com/vi/qiALzNWwzLU/hqdefault.jpg)](https://youtube.com/shorts/qiALzNWwzLU)
<video controls width="640">
  <source src="media/demo.mp4" type="video/mp4">
</video>


## Inspiration

We are hyped around the idea of emerging AI-Centered Companies. That was the main inspirations we had along these 40 hours.

Y Combinator "AI-First" company challenge inspired us to reimagine food ordering. We looked at DoorDash's end-to-end experience and asked: what if AI could predict what you want to eat before you even know it? We wanted to combine AI-powered personalization, data-driven UX, and conversational ordering into a single seamless experience where discovery and ordering happen in the same place.

## What it does

Haaangry is a TikTok-style food discovery and ordering app powered by Claude AI.

- Vertical video feed of cooking content with gesture-based navigation
- Swipe left → 6 AI-curated recipes (3 articles + 3 videos)
- Swipe right → 3 AI-matched restaurants with 3 recommended dishes each
- Clickable menu links that open real restaurant websites
- In-app ordering with cart and checkout
- Voice2Order and TextOrder AI assistants for ordering
- User profiles that track preferences, cravings, and order history over time

Claude analyzes each video's title, description, and tags to recommend contextually relevant restaurants and dishes in real-time.

## How we built it

### Backend
- FastAPI server with hot-reload and production-ready configuration
- Claude AI integration with strict JSON schema enforcement
- Parallel processing via ThreadPoolExecutor for recipe searches
- Pydantic 2.x for data validation and type safety
- CORS configured for iOS client requests
- Static file serving for locally hosted video assets
- noSQL-based storage: 603-line restaurants.json (20 restaurants × 20 menu items each)

### Frontend
- SwiftUI + MVVM architecture for clean separation of concerns
- AVFoundation for smooth 60fps video playback
- Async/Await networking with type-safe Codable models
- Gesture recognition for natural swipe navigation
- PlayerPool optimization to reuse AVPlayer instances and minimize memory
- Sheet-based modals for recommendations and recipes
- Clickable menu links using SwiftUI Link component
- Error recovery with retry mechanisms and graceful fallbacks

### Integration
- REST API (JSON over HTTP) between iOS app and FastAPI backend
- ThreadPoolExecutor for parallel Claude API calls (article + video recipe searches)
- End-to-end schema alignment ensuring menu URLs flow from backend to UI
- Graceful fallbacks on JSON parse errors or network issues

## Challenges we ran into

- Schema drift between Python (Pydantic) and Swift (Codable) models
- Duplicate struct definitions causing compilation errors
- SwiftUI View Builder constraints in computed properties
- Claude API rate limits during parallel requests
- JSON validation across a 603-line restaurant dataset
- Memory optimization for multiple concurrent video players
- Merge conflicts while coordinating parallel development
- Balancing technical sophistication with hackathon time constraints

## Accomplishments that we're proud of

- App that feels like something fundamentally novel
- Zero compilation errors in final codebase
- 100% menu URL coverage across all 20 restaurants
- Expanded catalog from 10 → 20 restaurants (400 total menu items)
- Sub-3-second average AI recommendation latency
- 60fps video playback on device
- End-to-end clickable menu links from backend to UI
- Production-ready backend running on port 8000 with strict validation
- 10 cuisine types (exceeded 8+ target)

## What we learned

### Product
- Balancing AI complexity with intuitive user experience
- Importance of gesture-based mobile navigation
- Value of immediate menu access via external links
- Trade-offs between feature breadth and implementation depth

### Technical
- FastAPI architecture and hot-reload workflow
- Claude AI JSON schema enforcement techniques
- AVFoundation video lifecycle management
- Pydantic validation and cross-platform data model alignment
- SwiftUI MVVM composition and async/await networking
- Parallel processing in Python with ThreadPoolExecutor


## What's next for haaangry

### Immediate Next Steps
- DoorDash API integration for live order placement
- Stripe / Apple Pay / Google Pay for payment processing
- AWS Maps / Google Routes API for delivery tracking
- Persistent order history and user accounts

### Medium-Term Enhancements
- Fine-tuned LLM or custom ML model for personalized recommendations
- Multi-turn conversation-to-order dialogue system
- User authentication with saved preferences
- Delivery-time-sensitive filtering based on location
- Real-time order tracking with push notifications

### Long-Term Vision
- Social features (share videos, reviews, ratings)
- Restaurant partnerships for exclusive deals
- Dietary filters (vegan, halal, allergies, kosher)
- Multi-language support for global expansion
- Analytics dashboard for engagement patterns

## Tech Stack

Backend: Python 3.10+, FastAPI 0.120.0, Anthropic Claude 3.5, Pydantic 2.12.3, Uvicorn  
Frontend: Swift 5+, SwiftUI (iOS 16+), AVFoundation, Combine  
AI/ML: Anthropic Claude API for semantic matching and recipe discovery  
Data: JSON storage (20 restaurants × 20 items, 226 videos)

## Key Metrics

- 226 videos in catalog
- 20 restaurants spanning 10 cuisines
- 400 menu items with full metadata
- 100% menu URL coverage
- ~3 second average Claude API response time
- 60fps video playback performance

## Demo Flow

1. Scroll through vertical video feed
2. Swipe left → 6 recipes (3 articles + 3 videos)
3. Swipe right → 3 AI-matched restaurants with 3 items each
4. Tap "View Full Menu" → opens restaurant website in Safari
5. Tap menu item → order confirmation with 30-minute ETA
6. Alternatively: voice or text chat with the AI assistant
