# Changelog

All notable changes to the Proxima Nexus OpenAPI specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2025-01-29

### Breaking changes

#### Path and parameter renames

- **User connections**
  - `GET/PUT/DELETE /user/{userId}/friends/{friendUserId}` → `PUT/DELETE /user/{userId}/connections/{targetUserId}` (path param `friendUserId` → `targetUserId`)
  - `GET /user/{userId}/friends` → `GET /user/{userId}/connections`
- **Event connections**
  - `PUT /event/{eventId}/attendees/{userId}` → `PUT /event/{eventId}/connection/{userId}`; DELETE added for removing attendees
  - `GET /event/{eventId}/attendees` → `GET /event/{eventId}/connections`
- **Group connections**
  - `PUT/DELETE /group/{groupId}/members/{userId}` → `PUT/DELETE /group/{groupId}/connection/{userId}`
  - `GET /group/{groupId}/members` → `GET /group/{groupId}/connections`

#### Operation ID renames

- `UserController_findOne` → `UserController_get`
- `UserController_addFriend` → `UserController_putConnection`
- `UserController_removeFriend` → `UserController_deleteConnection`
- `UserController_getFriends` → `UserController_getConnections`
- `EventController_findOne` → `EventController_get`
- `EventController_addAttendee` → `EventController_addConnection`
- `EventController_getAttendees` → `EventController_getConnections` (new `EventController_removeConnection` for DELETE)
- `GroupController_findOne` → `GroupController_get`
- `GroupController_addMember` → `GroupController_addConnection`
- `GroupController_removeMember` → `GroupController_removeConnection`
- `GroupController_getMembers` → `GroupController_getConnections`

#### Security

- All operations: security scheme `bearer` → `api_key`.

#### Request identity: header instead of body

- **Requester identity** is no longer sent in request bodies. Use the **`X-Proxima-Nexus-Requester-User-Id`** header on all relevant endpoints (create/update/delete, search, get-by-id, batch, list connections).
- **Removed** `requesterUserId` from: `CreateEntityDto`, `UpdateUserDto`, `CreateEventDto`, `UpdateEventDto`, `CreateGroupDto`, `UpdateGroupDto`.

#### Schema changes

- **Removed** `tenantId` from `UserDto`, `EventDto`, and `GroupDto`.
- **Added** `requesterConnection` (`EntityConnectionDto`) to `UserDto`, `EventDto`, and `GroupDto` for the calling user’s relationship to the entity.
- **EntityConnectionDto**: Now includes both `state` and `type`. New mutation DTOs: `MutateUserConnectionDto`, `MutateEventEntityConnectionDto`, `MutateGroupEntityConnectionDto` (with `type` only).
- **User connections**
  - PUT create/update: **Required** request body `MutateUserConnectionDto` with `type` (`friend` | `blocked`).
  - DELETE: **Required** query param `type` (`friend` | `blocked`); success response **200** → **204** (no body).
- **Event/group connections**
  - PUT: **Required** request body `MutateEventEntityConnectionDto` or `MutateGroupEntityConnectionDto` with `type`.
  - DELETE: **Required** request body (same DTO); success response **200** → **204** (no body).
- **Connection DTOs** (`UserEntityConnectionDto`, etc.): **Added** required `type`; **added** `blocked` to `state` enum.
- **Group type**: Now enum `open` | `request` | `invite` (replaces free-form string).
- **Events**: `CreateEventDto`/`EventDto`/`UpdateEventDto` add `maxNumAttendees`; `EventDto` adds `numAttendees`.
- **Groups**: `GroupDto` adds `numMembers` and `requesterConnection`.

### Added

- Optional `state` and `type` query filters on `GET /user/{userId}/connections`, `GET /event/{eventId}/connections`, and `GET /group/{groupId}/connections`.
- Explicit 400/401 response descriptions for create and visibility errors where applicable.
