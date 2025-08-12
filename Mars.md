Frontend (React)                    Backend (NestJS)
     │                                   │
     │ 1. User logs in via Supabase      │
     │    supabase.auth.signInWith...    │
     │                                   │
     │ 2. Get access token               │
     │    session.access_token           │
     │                                   │
     │ 3. Make API request with token    │
     │ ──────────────────────────────────┼──► HTTP Request
     │   Authorization: Bearer <token>   │    Headers: { Authorization: "Bearer xyz..." }
     │                                   │
     │                                   │ 4. Request hits controller
     │                                   │    @UseGuards(JwtAuthGuard)
     │                                   │    ▼
     │                                   │ 5. JwtAuthGuard.canActivate()
     │                                   │    • Extract token from header
     │                                   │    • Call authService.validateToken()
     │                                   │    ▼
     │                                   │ 6. AuthService.validateToken()
     │                                   │    • Call supabase.auth.getUser(token)
     │                                   │    • Verify token with Supabase
     │                                   │    ▼
     │                                   │ 7. If valid: attach user to request
     │                                   │    request['user'] = user
     │                                   │    ▼
     │                                   │ 8. Controller method executes
     │                                   │    @CurrentUser() gets user from request
     │                                   │    ▼
     │ ◄──────────────────────────────────┼──── 9. Return response
     │   { id: "...", email: "..." }     │