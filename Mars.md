# Bitwise-Server - UML Class Diagrams

This document contains comprehensive UML class diagrams for all six features in the Bitwise-Server project with proper class relationships including inheritance, composition, dependency, and interfaces.

**Note:** Use PlantUML to render these diagrams. Copy the PlantUML code into [PlantUML Online Editor](http://www.plantuml.com/plantuml/uml/)

---

## Table of Contents

1. [User Registration](#1-user-registration)
2. [Edit Information](#2-edit-information)
3. [Navigation Tools](#3-navigation-tools)
4. [Boolean Calculator](#4-boolean-calculator)
5. [Karnaugh Map](#5-karnaugh-map)
6. [Circuit Diagram](#6-circuit-diagram)

---

## 1. User Registration

**Status:** ✅ Implemented  
**Files:** `src/auth/*`  
**Dependencies:** Supabase, Passport.js, JWT

### PlantUML Class Diagram

```plantuml
@startuml User Registration
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface IAuthGuard {
  {abstract} +canActivate(context: ExecutionContext): boolean | Promise<boolean>
}

interface IAuthStrategy {
  {abstract} +validate(payload: any): Promise<User>
}

class AuthController {
  -authService: AuthService
  
  +signup(signupDto: SignupDto): Promise<ApiResponse>
  +signin(signinDto: SigninDto): Promise<ApiResponse>
  +getProfile(user: User): ProfileDto
  +verifyToken(user: User): VerificationResult
}

class AuthService {
  -supabase: SupabaseClient
  -configService: ConfigService
  
  -initializeSupabase(): void
  +signup(signupDto: SignupDto): Promise<User>
  +signin(signinDto: SigninDto): Promise<AuthToken>
  +validateToken(token: string): Promise<User>
  +validateUser(userId: string): Promise<User | null>
}

class JwtAuthGuard implements IAuthGuard {
  +canActivate(context: ExecutionContext): boolean | Promise<boolean>
}

class JwtStrategy implements IAuthStrategy {
  -configService: ConfigService
  
  +validate(payload: JwtPayload): Promise<User>
}

class CurrentUserDecorator {
  {static} +getUser(context: ExecutionContext): User
}

class SignupDto {
  +email: string
  +password: string
  +firstName?: string
  +lastName?: string
}

class SigninDto {
  +email: string
  +password: string
}

class AuthToken {
  +accessToken: string
  +refreshToken?: string
  +expiresIn: number
}

class SupabaseClient {
  +auth.signUp(email, password): Promise<User>
  +auth.signInWithPassword(email, password): Promise<Session>
  +auth.getUser(token): Promise<User>
  +auth.admin.getUserById(id): Promise<User>
}

class User {
  -id: string
  -email: string
  -user_metadata: object
  -created_at: Date
  -updated_at: Date
}

AuthController --> AuthService : uses
AuthController --> JwtAuthGuard : uses
AuthController --> CurrentUserDecorator : uses
AuthService --> SupabaseClient : delegates to
JwtAuthGuard --> JwtStrategy : validates with
JwtStrategy --> AuthService : validates user
CurrentUserDecorator --> AuthService : extracts user
AuthService --> User : returns
AuthService --> AuthToken : returns
AuthController --> SignupDto : receives
AuthController --> SigninDto : receives

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **uses** | AuthController | AuthService | Controller depends on service |
| **delegates to** | AuthService | SupabaseClient | Service delegates auth to external provider |
| **implements** | JwtAuthGuard | IAuthGuard | Guard implements authentication interface |
| **validates with** | JwtAuthGuard | JwtStrategy | Guard uses strategy to validate tokens |
| **returns** | AuthService | User | Service returns user object after authentication |

### Methods & Responsibilities

**AuthController:**
- Routes HTTP requests to appropriate service methods
- Extracts DTOs from request bodies
- Returns standardized API responses

**AuthService:**
- Initializes Supabase client with configuration
- Handles signup/signin logic
- Validates JWT tokens
- Returns authenticated users

**JwtAuthGuard:**
- Protects routes with @UseGuards(JwtAuthGuard)
- Validates JWT presence in request headers
- Delegates to JwtStrategy for token validation

**JwtStrategy:**
- Decodes JWT payload
- Validates token signature
- Calls AuthService.validateUser()

---

## 2. Edit Information (User Profile)

**Status:** ⚠️ Partially Implemented  
**Files:** `src/user/*`, `src/auth/*`  
**Dependencies:** Prisma ORM, bcrypt

### PlantUML Class Diagram

```plantuml
@startuml Edit Information
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface IUserService {
  {abstract} +getProfile(userId: string): Promise<ProfileDto>
  {abstract} +updateProfile(userId: string, dto: UpdateProfileDto): Promise<User>
  {abstract} +changePassword(userId: string, oldPassword: string, newPassword: string): Promise<boolean>
}

interface IAuditService {
  {abstract} +recordChange(userId: string, changeType: string, data: any): Promise<void>
}

class UserController {
  -userService: UserService
  -auditService: AuditService
  
  +getProfile(userId: string): Promise<ProfileDto>
  +updateProfile(userId: string, dto: UpdateProfileDto): Promise<ApiResponse>
  +changePassword(userId: string, dto: ChangePasswordDto): Promise<ApiResponse>
}

class UserService implements IUserService {
  -prisma: PrismaService
  -passwordService: PasswordService
  -auditService: AuditService
  
  +getProfile(userId: string): Promise<ProfileDto>
  +updateProfile(userId: string, dto: UpdateProfileDto): Promise<User>
  +changePassword(userId: string, oldPassword: string, newPassword: string): Promise<boolean>
  -validatePasswordStrength(password: string): boolean
}

class UserRepository {
  -prisma: PrismaService
  
  +findById(id: string): Promise<User | null>
  +save(user: User): Promise<User>
  +updateProfile(id: string, data: object): Promise<User>
  +updatePassword(id: string, hash: string): Promise<User>
}

class PasswordService {
  -bcrypt: BcryptModule
  
  +hashPassword(plainPassword: string): Promise<string>
  +validatePassword(plain: string, hash: string): Promise<boolean>
  +generateTemporaryPassword(): string
}

class AuditService implements IAuditService {
  -prisma: PrismaService
  
  +recordChange(userId: string, changeType: string, data: any): Promise<void>
  +recordPasswordChange(userId: string): Promise<void>
  +recordProfileChange(userId: string, oldData: object, newData: object): Promise<void>
  +getAuditLog(userId: string): Promise<AuditLog[]>
}

class PrismaService {
  +user.findUnique(where: object): Promise<User | null>
  +user.update(where: object, data: object): Promise<User>
  +auditLog.create(data: object): Promise<AuditLog>
}

class User {
  -id: string
  -email: string
  -password_hash: string
  -first_name: string
  -last_name: string
  -avatar_url?: string
  -bio?: string
  -created_at: Date
  -updated_at: Date
}

class ProfileDto {
  +id: string
  +email: string
  +firstName: string
  +lastName: string
  +avatarUrl?: string
  +bio?: string
}

class UpdateProfileDto {
  +firstName?: string
  +lastName?: string
  +avatarUrl?: string
  +bio?: string
}

class ChangePasswordDto {
  +oldPassword: string
  +newPassword: string
  +confirmPassword: string
}

class AuditLog {
  -id: string
  -userId: string
  -changeType: string
  -data: object
  -createdAt: Date
}

UserController --> UserService : uses
UserController --> AuditService : uses
UserService --> UserRepository : persists via
UserService --> PasswordService : hashes with
UserService --> AuditService : records to
UserRepository --> PrismaService : queries with
UserRepository --> User : maps to/from
PrismaService --> User : persists
UserService --> ProfileDto : returns
UserController --> UpdateProfileDto : receives
UserController --> ChangePasswordDto : receives
AuditService --> PrismaService : logs to
AuditService --> AuditLog : creates

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **implements** | UserService | IUserService | Service implements the interface contract |
| **uses** | UserController | UserService | Controller depends on service |
| **persists via** | UserService | UserRepository | Service uses repository pattern |
| **hashes with** | UserService | PasswordService | Service delegates password hashing |
| **queries with** | UserRepository | PrismaService | Repository uses ORM for DB access |
| **maps to/from** | UserRepository | User | Repository maps data to domain model |

### Methods & Responsibilities

**UserController:**
- HTTP GET /api/user/:userId → retrieves profile
- HTTP PATCH /api/user/:userId → updates profile
- HTTP POST /api/user/change-password → changes password

**UserService:**
- Implements business logic for user operations
- Validates password strength and changes
- Triggers audit logging on changes

**UserRepository:**
- Database access layer
- Single responsibility: query/persist users

**PasswordService:**
- Bcrypt password hashing
- Password validation with salt verification

**AuditService:**
- Logs all user profile changes
- Maintains audit trail for compliance

---

## 3. Navigation Tools

**Status:** ❌ Not Implemented  
**Recommended Files:** `src/navigation/*`  
**Dependencies:** None (core framework)

### PlantUML Class Diagram

```plantuml
@startuml Navigation Tools
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface INavigationService {
  {abstract} +buildMenuForUser(user: User): Menu
  {abstract} +getBreadcrumbs(route: string): Breadcrumb[]
  {abstract} +registerNavItem(item: NavItem): void
}

interface IPermissionService {
  {abstract} +canAccess(user: User, resource: string): boolean
  {abstract} +checkRoute(userId: string, route: string): Promise<boolean>
}

class NavigationController {
  -navigationService: NavigationService
  -authService: AuthService
  
  +getMenu(userId: string): Promise<Menu>
  +getBreadcrumbs(route: string): Promise<Breadcrumb[]>
  +getNavigation(userId: string): Promise<NavigationData>
}

class NavigationService implements INavigationService {
  -menuItems: Map<string, NavItem>
  -permissionService: PermissionService
  -prisma: PrismaService
  
  +buildMenuForUser(user: User): Menu
  +registerNavItem(item: NavItem): void
  +getBreadcrumbs(route: string): Breadcrumb[]
  +getMenuHierarchy(): NavItem[]
  -filterMenuByPermissions(items: NavItem[], user: User): NavItem[]
}

class PermissionService implements IPermissionService {
  -prisma: PrismaService
  
  +canAccess(user: User, resource: string): boolean
  +checkRoute(userId: string, route: string): Promise<boolean>
  +getUserRoles(userId: string): Promise<Role[]>
  +getRolePermissions(roleId: string): Promise<Permission[]>
}

class NavItem {
  -id: string
  -label: string
  -route: string
  -icon?: string
  -order: number
  -visible: boolean
  -requiredPermission?: string
  -children: NavItem[]
  
  +addChild(item: NavItem): void
  +getChildren(): NavItem[]
}

class Menu {
  -items: NavItem[]
  -userId: string
  
  +addItem(item: NavItem): void
  +getItems(): NavItem[]
}

class Breadcrumb {
  -label: string
  -route: string
  -active: boolean
  -icon?: string
}

class NavigationData {
  +menu: Menu
  +breadcrumbs: Breadcrumb[]
  +currentRoute: string
}

class User {
  -id: string
  -email: string
  -roles: Role[]
}

class Role {
  -id: string
  -name: string
  -permissions: Permission[]
}

class Permission {
  -id: string
  -name: string
  -resource: string
  -action: string
}

class AuthService {
  +validateUser(userId: string): Promise<User>
}

class PrismaService {
  +user.findUnique(where: object)
  +role.findMany(where: object)
  +permission.findMany(where: object)
}

NavigationController --> NavigationService : uses
NavigationController --> AuthService : validates with
NavigationService --> PermissionService : checks access via
NavigationService --> PrismaService : queries with
PermissionService --> PrismaService : queries with
NavigationService "1" --> "*" NavItem : composes
NavItem "1" --> "*" NavItem : nests
Menu "1" --> "*" NavItem : contains
NavigationService --> Menu : builds
NavigationService --> Breadcrumb : generates
PermissionService --> User : checks
PermissionService --> Role : verifies
PermissionService --> Permission : evaluates

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **implements** | NavigationService | INavigationService | Service implements contract |
| **uses** | NavigationController | NavigationService | Controller depends on service |
| **checks access via** | NavigationService | PermissionService | Service delegates permission checks |
| **composes** | NavigationService | NavItem | Service aggregates menu items |
| **nests** | NavItem | NavItem | Hierarchical menu structure |
| **contains** | Menu | NavItem | Menu is container for items |

### Methods & Responsibilities

**NavigationController:**
- HTTP GET /api/navigation/menu/:userId → retrieves user's menu
- HTTP GET /api/navigation/breadcrumbs → gets breadcrumb trail
- HTTP GET /api/navigation/full → complete navigation data

**NavigationService:**
- Builds dynamic menus based on user permissions
- Maintains navigation item registry
- Generates breadcrumb trails for routes

**PermissionService:**
- Checks if user has access to resources
- Validates route permissions
- Retrieves user roles and permissions

**NavItem:**
- Represents individual menu item
- Supports nested hierarchies
- Stores permission requirements

---

## 4. Boolean Calculator

**Status:** ✅ Implemented  
**Files:** `src/calculator/*`  
**Dependencies:** JavaScript VM module

### PlantUML Class Diagram

```plantuml
@startuml Boolean Calculator
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface ICalculatorService {
  {abstract} +simplifyExpression(expr: string): Promise<CalculationResponse>
  {abstract} +evaluateExpression(expr: string, assignments: object): Promise<CalculationResponse>
  {abstract} +generateTruthTable(expr: string): Promise<CalculationResponse>
}

class CalculatorController {
  -calculatorService: CalculatorService
  
  +simplify(simplifyDto: SimplifyExpressionDto): Promise<CalculationResponse>
  +evaluate(evaluateDto: EvaluateExpressionDto): Promise<CalculationResponse>
  +truthTable(truthTableDto: TruthTableDto): Promise<CalculationResponse>
}

class CalculatorService implements ICalculatorService {
  -jsContext: VMContext
  
  -initializeJavaScriptContext(): void
  -sanitizeExpression(expression: string): string
  +simplifyExpression(expr: string): Promise<CalculationResponse>
  +evaluateExpression(expr: string, assignments: object): Promise<CalculationResponse>
  +generateTruthTable(expr: string): Promise<CalculationResponse>
  +calculate(expression: string): Promise<CalculationResponse>
}

abstract class ExprNode {
  {abstract} +evaluate(env: Map): boolean
  {abstract} +toString(): string
  {abstract} +equals(obj: ExprNode): boolean
  {abstract} +clone(): ExprNode
}

class VarNode extends ExprNode {
  -variableName: string
  
  +evaluate(env: Map): boolean
  +toString(): string
}

class TrueNode extends ExprNode {
  +evaluate(env: Map): boolean
  +toString(): string
}

class FalseNode extends ExprNode {
  +evaluate(env: Map): boolean
  +toString(): string
}

class NotExpression extends ExprNode {
  -subs: ExprNode[]
  
  +evaluate(env: Map): boolean
  +toString(): string
}

class AndExpression extends ExprNode {
  -subs: ExprNode[]
  
  +evaluate(env: Map): boolean
  +toString(): string
  +contains(obj: ExprNode): boolean
}

class OrExpression extends ExprNode {
  -subs: ExprNode[]
  
  +evaluate(env: Map): boolean
  +toString(): string
  +contains(obj: ExprNode): boolean
}

class Parser {
  -NOT_EXPRESSIONS: string[]
  -AND_EXPRESSIONS: string[]
  -OR_EXPRESSIONS: string[]
  
  +parse(expression: string): ExprNode
  +standardize(expression: string): string
  -parseStandard(std: string): string[]
  -processParsedArray(array: string[]): ExprNode
}

class VariableManager {
  -variables: Variable[]
  
  +get(varName: string): Variable
  +getVariables(): Variable[]
  +getTruthAssignments(): object[]
  +clear(): void
}

class EquivalencyLaws {
  -laws: Function[]
  
  +simplify(expression: ExprNode): Step[]
  -applyLawOnce(expr: ExprNode, law: Function): ExprNode | false
  -commutative(expr: ExprNode): ExprNode | false
  -associative(expr: ExprNode): ExprNode | false
  -distributive(expr: ExprNode): ExprNode | false
  -identity(expr: ExprNode): ExprNode | false
  -deMorgans(expr: ExprNode): ExprNode | false
}

class Step {
  -result: string
  -law: string
  -extraData?: any
}

class SimplifyExpressionDto {
  +expression: string
}

class CalculationResponse {
  +success: boolean
  +result?: any
  +error?: string
}

class TruthTable {
  -variables: string[]
  -table: object[]
  
  +getRow(index: number): object
  +getColumn(variable: string): boolean[]
}

class CalculationScript {
  -defaultExpression: string
  -steps: Step[]
}

CalculatorController --> CalculatorService : uses
CalculatorService --> Parser : uses
CalculatorService --> EquivalencyLaws : uses
Parser --> ExprNode : builds
Parser --> VariableManager : manages
EquivalencyLaws --> ExprNode : transforms
ExprNode <|-- VarNode
ExprNode <|-- TrueNode
ExprNode <|-- FalseNode
ExprNode <|-- NotExpression
ExprNode <|-- AndExpression
ExprNode <|-- OrExpression
CalculatorService --> CalculationResponse : returns
CalculatorService --> TruthTable : generates
EquivalencyLaws --> Step : creates
CalculatorService --> CalculationScript : returns

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **implements** | CalculatorService | ICalculatorService | Service implements contract |
| **uses** | CalculatorService | Parser | Service uses parser for AST building |
| **builds** | Parser | ExprNode | Parser constructs expression tree |
| **extends** | VarNode | ExprNode | Specialization of abstract expression |
| **transforms** | EquivalencyLaws | ExprNode | Laws convert expressions |
| **generates** | CalculatorService | TruthTable | Service produces truth tables |

### Methods & Responsibilities

**CalculatorController:**
- HTTP POST /api/calculator/simplify → simplifies Boolean expression
- HTTP POST /api/calculator/evaluate → evaluates with assignments
- HTTP POST /api/calculator/truth-table → generates truth table

**CalculatorService:**
- Initializes JavaScript VM context with Boolean algebra engine
- Sanitizes input expressions
- Coordinates parsing and simplification
- Generates truth tables

**Parser:**
- Parses string expressions into AST
- Tokenizes and standardizes notation
- Builds expression node tree

**ExprNode (Abstract):**
- Base class for all Boolean expressions
- Defines evaluation and string representation

**VarNode, TrueNode, FalseNode:**
- Leaf nodes of expression tree
- Base expressions that can be combined

**NotExpression, AndExpression, OrExpression:**
- Operator nodes
- Contain sub-expressions
- Support Boolean operations

**EquivalencyLaws:**
- Implements Boolean algebra simplification laws
- Applies laws iteratively until minimal form
- Laws: commutative, associative, distributive, absorption, De Morgan's, etc.

---

## 5. Karnaugh Map

**Status:** ❌ Not Implemented  
**Recommended Files:** `src/karnaugh/*`  
**Dependencies:** Calculator service (parser)

### PlantUML Class Diagram

```plantuml
@startuml Karnaugh Map
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface IKarnaughService {
  {abstract} +buildMap(expr: ExprNode, vars: string[]): KarnaughMap
  {abstract} +findGroups(map: KarnaughMap): Grouping[]
  {abstract} +minimizeExpression(groups: Grouping[]): MinimizedExpression
}

class KarnaughController {
  -karnaughService: KarnaughService
  -calculatorService: CalculatorService
  
  +fromExpression(expr: string, vars: string[]): Promise<KarnaughResult>
  +visualize(mapId: string): Promise<Diagram>
  +minimize(mapData: KarnaughData): Promise<MinimizedExpression>
}

class KarnaughService implements IKarnaughService {
  -calculatorService: CalculatorService
  -maps: Map<string, KarnaughMap>
  
  +buildMap(expr: ExprNode, vars: string[]): KarnaughMap
  +findGroups(map: KarnaughMap): Grouping[]
  +minimizeExpression(groups: Grouping[]): MinimizedExpression
  +generateVisualization(map: KarnaughMap): Diagram
  -generateMapId(): string
  -calculateMapDimensions(numVars: number): {rows: number, cols: number}
}

class KarnaughMap {
  -mapId: string
  -variables: string[]
  -numVars: number
  -rows: number
  -cols: number
  -cells: Cell[][]
  
  +getCell(row: number, col: number): Cell
  +setValue(row: number, col: number, value: 0|1|X): void
  +buildFromTruthTable(table: TruthTable): void
  +validate(): boolean
}

class Cell {
  -row: number
  -col: number
  -value: 0 | 1 | X
  -covered: boolean
  -groupId?: string
  -minterms: number[]
  
  +setValue(value: 0|1|X): void
  +getMintermIndex(): number
}

class Grouping {
  -groupId: string
  -cells: Cell[]
  -size: number
  -type: 'prime' | 'essential'
  
  +toPrimeImplicant(): string
  +toMinimalTerm(): string
  +getSize(): number
  +getCells(): Cell[]
}

class MinimizedExpression {
  -sop: string
  -pos: string
  -expression: string
  -variables: string[]
  -groupings: Grouping[]
  
  +toString(): string
  +getSOPTerms(): string[]
  +getPOSTerms(): string[]
}

class Diagram {
  -format: 'svg' | 'png' | 'canvas'
  -content: string | Blob
  -mapId: string
  
  +render(): HTMLElement
  +export(format: string): Blob
}

class KarnaughResult {
  +mapId: string
  +mapData: KarnaughMap
  +minimizedExpression: MinimizedExpression
  +groupings: Grouping[]
}

class KarnaughData {
  +mapId: string
  +cells: object[]
  +variables: string[]
}

class CalculatorService {
  +parseExpression(expr: string): ExprNode
  +generateTruthTable(expr: string): TruthTable
}

class ExprNode {
  +evaluate(env: Map): boolean
}

class TruthTable {
  -variables: string[]
  -table: object[]
}

KarnaughController --> KarnaughService : uses
KarnaughController --> CalculatorService : uses
KarnaughService --> KarnaughMap : builds
KarnaughService --> ExprNode : parses
KarnaughService --> Grouping : finds optimal
KarnaughService --> MinimizedExpression : generates
KarnaughService --> Diagram : creates
KarnaughService --> TruthTable : uses
KarnaughMap "1" --> "*" Cell : contains
Grouping "1" --> "*" Cell : covers
MinimizedExpression "1" --> "*" Grouping : includes
KarnaughResult --> KarnaughMap : references
KarnaughResult --> MinimizedExpression : references
KarnaughResult --> Grouping : includes

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **implements** | KarnaughService | IKarnaughService | Service implements contract |
| **uses** | KarnaughService | KarnaughMap | Service manages map instances |
| **builds** | KarnaughService | KarnaughMap | Service creates K-map structure |
| **contains** | KarnaughMap | Cell | Map is aggregation of cells |
| **covers** | Grouping | Cell | Grouping aggregates cells |
| **includes** | MinimizedExpression | Grouping | Result contains groupings |

### Methods & Responsibilities

**KarnaughController:**
- HTTP POST /api/karnaugh/build → creates K-map from expression
- HTTP GET /api/karnaugh/visualize/:mapId → renders diagram
- HTTP POST /api/karnaugh/minimize → finds minimized expression

**KarnaughService:**
- Converts Boolean expressions to K-maps
- Finds optimal cell groupings (prime implicants)
- Generates minimized SOP/POS expressions
- Creates SVG visualizations

**KarnaughMap:**
- Maintains 2D grid of cells (2-var to 5-var support)
- Maps truth values to grid positions
- Calculates row/column labels based on Gray code

**Cell:**
- Represents single K-map cell
- Stores value (0, 1, or don't-care)
- Tracks group membership

**Grouping:**
- Represents group of adjacent cells (1, 2, 4, 8, 16...)
- Converts to Boolean term (SOP/POS)
- Marked as essential or prime

**MinimizedExpression:**
- Result of K-map minimization
- Stores both SOP and POS forms
- Includes all groupings used

---

## 6. Circuit Diagram

**Status:** ❌ Not Implemented  
**Recommended Files:** `src/circuit/*`  
**Dependencies:** None (core framework)

### PlantUML Class Diagram

```plantuml
@startuml Circuit Diagram
!theme plain
skinparam backgroundColor #FEFEFE
skinparam classBackgroundColor #F0F0F0

interface ICircuitService {
  {abstract} +addComponent(component: Component): string
  {abstract} +connect(from: Pin, to: Pin): void
  {abstract} +simulate(inputs: object): SimulationResult
  {abstract} +exportNetlist(): Netlist
}

interface IComponent {
  {abstract} -id: string
  {abstract} +evaluate(): boolean | object
  {abstract} +getPins(): Pin[]
}

class CircuitController {
  -circuitService: CircuitService
  
  +addComponent(component: ComponentDto): Promise<string>
  +connectPins(from: PinReference, to: PinReference): Promise<void>
  +simulate(inputs: object): Promise<SimulationResult>
  +exportNetlist(): Promise<Netlist>
  +getCircuitDiagram(): Promise<Diagram>
}

class CircuitService implements ICircuitService {
  -components: Map<string, Component>
  -wires: Wire[]
  -netlist: Netlist
  -simulator: Simulator
  
  +addComponent(component: Component): string
  +connect(from: Pin, to: Pin): void
  +removeComponent(componentId: string): void
  +simulate(inputs: object): SimulationResult
  +layout(): Layout
  +exportNetlist(): Netlist
  -validateCircuit(): boolean
  -checkForCycles(): boolean
}

abstract class Component implements IComponent {
  -id: string
  -label: string
  -inputs: Pin[]
  -outputs: Pin[]
  -position?: {x: number, y: number}
  
  {abstract} +evaluate(): boolean | object
  +getPins(): Pin[]
  +getInputs(): Pin[]
  +getOutputs(): Pin[]
}

class ANDGate extends Component {
  -inputCount: 2 | 3 | 4 | 8
  
  +evaluate(): boolean
}

class ORGate extends Component {
  -inputCount: 2 | 3 | 4 | 8
  
  +evaluate(): boolean
}

class NOTGate extends Component {
  +evaluate(): boolean
}

class NANDGate extends Component {
  +evaluate(): boolean
}

class NORGate extends Component {
  +evaluate(): boolean
}

class XORGate extends Component {
  +evaluate(): boolean
}

class XNORGate extends Component {
  +evaluate(): boolean
}

class InputNode extends Component {
  -name: string
  -value: boolean
  
  +setValue(value: boolean): void
  +evaluate(): boolean
}

class OutputNode extends Component {
  -name: string
  -value: boolean
  
  +evaluate(): boolean
}

class Pin {
  -id: string
  -name: string
  -type: 'input' | 'output'
  -componentId: string
  -value: boolean
  -connectedTo?: Pin
  
  +connect(pin: Pin): void
  +disconnect(): void
  +getValue(): boolean
  +setValue(value: boolean): void
}

class Wire {
  -id: string
  -from: Pin
  -to: Pin
  -signal: boolean
  -label?: string
  
  +propagateSignal(): void
  +getSignal(): boolean
}

class Netlist {
  -id: string
  -name: string
  -components: Component[]
  -wires: Wire[]
  -inputs: InputNode[]
  -outputs: OutputNode[]
  
  +addComponent(component: Component): void
  +addWire(wire: Wire): void
  +toJSON(): object
  +toVHDL(): string
  +toVerilog(): string
  +validate(): boolean
}

class Simulator {
  -netlist: Netlist
  -maxIterations: number
  
  +run(inputs: object): SimulationResult
  -evaluateComponent(comp: Component): void
  -propagateSignals(): void
  -detectCycles(): boolean
}

class SimulationResult {
  -outputs: object
  -signals: Map<string, boolean>
  -traces: object
  -executionTime: number
  
  +getOutput(name: string): boolean
  +getSignal(pinId: string): boolean
}

class Layout {
  -components: Map<string, {x: number, y: number}>
  
  +getComponentPosition(id: string): {x: number, y: number}
  +setComponentPosition(id: string, x: number, y: number): void
  +autoLayout(): void
}

class Diagram {
  -format: 'svg' | 'png' | 'canvas'
  -content: string | Blob
  -layout: Layout
  
  +render(): HTMLElement
  +export(format: string): Blob
}

class Renderer {
  +render(netlist: Netlist, layout: Layout): Diagram
  +layout(netlist: Netlist): Layout
  -drawComponent(comp: Component): void
  -drawWire(wire: Wire): void
}

class ComponentDto {
  +type: 'AND' | 'OR' | 'NOT' | 'NAND' | 'NOR' | 'XOR' | 'XNOR' | 'INPUT' | 'OUTPUT'
  +label: string
  +position?: {x: number, y: number}
}

class PinReference {
  +componentId: string
  +pinId: string
}

CircuitController --> CircuitService : manages
CircuitController --> ComponentDto : receives
CircuitService --> Component : creates
CircuitService --> Netlist : maintains
CircuitService --> Simulator : uses
CircuitService --> Renderer : uses
Component <|-- ANDGate
Component <|-- ORGate
Component <|-- NOTGate
Component <|-- NANDGate
Component <|-- NORGate
Component <|-- XORGate
Component <|-- XNORGate
Component <|-- InputNode
Component <|-- OutputNode
Component "1" --> "*" Pin : has
Netlist "1" --> "*" Component : contains
Netlist "1" --> "*" Wire : contains
Wire --> Pin : connects
Simulator --> Netlist : evaluates
Simulator --> Component : evaluates
Renderer --> Netlist : visualizes
Renderer --> Layout : uses

@enduml
```

### Key Relationships

| Relationship Type | From | To | Meaning |
|---|---|---|---|
| **implements** | CircuitService | ICircuitService | Service implements contract |
| **extends** | ANDGate | Component | Specific gate type specialization |
| **manages** | CircuitController | CircuitService | Controller orchestrates |
| **creates** | CircuitService | Component | Service factory for gates |
| **contains** | Netlist | Component | Netlist is container |
| **contains** | Netlist | Wire | Netlist holds connections |
| **connects** | Wire | Pin | Wire links pins |
| **has** | Component | Pin | Component owns pins |
| **uses** | CircuitService | Simulator | Service delegates execution |
| **uses** | CircuitService | Renderer | Service creates visualizations |

### Methods & Responsibilities

**CircuitController:**
- HTTP POST /api/circuit/add-component → adds gate/input/output
- HTTP POST /api/circuit/connect → creates wire between pins
- HTTP POST /api/circuit/simulate → runs simulation
- HTTP GET /api/circuit/export → exports netlist
- HTTP GET /api/circuit/diagram → renders SVG/canvas

**CircuitService:**
- Manages component and wire collections
- Validates circuit topology (no cycles, all connections)
- Coordinates simulation execution
- Exports netlist to VHDL/Verilog

**Component (Abstract):**
- Base class for all logic elements
- Stores pins (inputs/outputs)
- Defines evaluation behavior

**Gate Classes (ANDGate, ORGate, etc.):**
- Implement specific Boolean logic
- Accept variable input counts
- Return single boolean output

**InputNode / OutputNode:**
- User-controllable inputs
- Circuit output collectors
- Special component types

**Pin:**
- Represents connection point
- Stores current signal value
- Maintains connection to other pins

**Wire:**
- Represents physical connection
- Transfers signal between pins
- Labels can annotate signals

**Netlist:**
- Complete circuit representation
- Serializable to JSON/VHDL/Verilog
- Validates connectivity and logic

**Simulator:**
- Event-driven signal propagation
- Evaluates components iteratively
- Detects combinatorial vs sequential behavior

**Renderer:**
- Converts netlist to visual diagram
- Positions components and routes wires
- Exports to SVG or raster formats

---

## Summary Comparison Table

| Feature | Status | Inheritance | Composition | Interfaces | Key Relationships |
|---------|--------|-------------|-------------|-----------|---|
| **User Registration** | ✅ | N/A | AuthService→SupabaseClient | IAuthGuard, IAuthStrategy | uses, delegates, implements |
| **Edit Information** | ⚠️ | N/A | UserService→UserRepository→PrismaService | IUserService, IAuditService | uses, implements, persists |
| **Navigation Tools** | ❌ | N/A | NavItem→NavItem (hierarchy) | INavigationService, IPermissionService | implements, nests, composes |
| **Boolean Calculator** | ✅ | ExprNode extends (VarNode, TrueNode, etc.) | CalculatorService→Parser→ExprNode | ICalculatorService | extends, uses, builds, transforms |
| **Karnaugh Map** | ❌ | N/A | KarnaughService→KarnaughMap→Cell | IKarnaughService | builds, contains, covers |
| **Circuit Diagram** | ❌ | Component extends (ANDGate, ORGate, etc.) | CircuitService→Component, Wire, Netlist | ICircuitService, IComponent | extends, contains, implements |

---

## Design Patterns Used

### 1. **Service Layer Pattern**
All features use a layered architecture:
```
Controller → Service → Repository/External Service → Database/API
```

### 2. **Dependency Injection**
NestJS provides constructor-based DI:
```typescript
constructor(private service: ServiceClass) {}
```

### 3. **Repository Pattern**
Data access layer separates business logic from persistence:
```typescript
// UserService uses UserRepository
// Repository uses PrismaService
// This isolates DB changes from business logic
```

### 4. **Strategy Pattern**
JWT Authentication uses strategies:
```
JwtAuthGuard uses JwtStrategy for token validation
Multiple strategies (OAuth, API Key, etc.) can swap in
```

### 5. **Factory Pattern**
Circuit service creates components:
```typescript
const gate = circuitService.addComponent(new ANDGate())
```

### 6. **Composite Pattern**
Navigation uses hierarchical structure:
```
Menu contains NavItem[]
NavItem contains NavItem[] (children)
```

### 7. **Visitor Pattern**
Boolean calculator uses law application:
```
EquivalencyLaws visits each ExprNode
Applies transformations recursively
```

### 8. **Observer Pattern**
Circuit simulation uses signal propagation:
```
Wire propagates signal to connected Pin
Pin notifies dependent components
```

---

## Relationship Legend

| Symbol | Meaning |
|--------|---------|
| `→` | uses / depends on |
| `→\|` | inherits from (extends) |
| `◇→` | contains (composition) |
| `◇--→` | aggregates (weak composition) |
| `..→` | implements interface |
| `--→` | associates with |

---

## Export & Usage

### To Render in PlantUML Online:
1. Copy any PlantUML code block
2. Visit http://www.plantuml.com/plantuml/uml/
3. Paste and render

### To Generate PNG locally:
```bash
# Install PlantUML
npm install -g plantuml

# Generate PNG
plantuml CLASS_DIAGRAMS_UML.md -o ../diagrams/

# Or use Docker
docker run -v $(pwd):/workspace -w /workspace plantuml/plantuml:latest -png CLASS_DIAGRAMS_UML.md
```

### To use in documentation:
- Export as SVG for web
- Export as PNG for presentations
- Keep .md for version control

---

**Last Updated:** October 20, 2025  
**Project:** Bitwise-Server  
**Repository:** One-Team-One-Goal/bitwise-server  
**Branch:** development

