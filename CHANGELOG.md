# Changelog

All notable changes to the Proxima Nexus OpenAPI specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.3.2] - 2026-03-19

### Added

- **Geo-aware discovery filters**
  - `GET /user/{userId}/groups`: added optional `latitude` and `longitude` query params for nearby group discovery.
  - `GET /user/{userId}/events`: added optional `latitude` and `longitude` query params for nearby event discovery.

### Changed

- **Connection types**
  - Added `nearby` to connection `type` enums used by event, group, and event-series connection DTOs/responses and related filters.
  - `GetEventsDto` and group connection-type filters now accept `nearby` as a valid value.

## [2.3.1] - 2026-03-04

### Changed

- **Connection types**
  - Added `associated_group` to connection `type` enums across event, group, and event-series connection DTOs and related responses, so clients can distinguish when a group is linked to an event or series as an associated organizer rather than just an attendee/admin/owner.
  - `GetEventsDto` and other connection-type-based filters now accept `associated_group` as a valid value.

## [2.3.0] - 2026-03-04

### Added

- **Group event series**
  - `GET /group/{groupId}/event-series`: list event series associated with a group, returned as `EventSeriesEntityConnectionDto` objects (including connection `type`, `state`, and `seriesId`).
- **Event series batch lookup**
  - `POST /event-series/batch`: fetch multiple event series in a single call using `GetEventSeriesDto.seriesIds`.
- **Schemas**
  - `EventSeriesEntityConnectionDto`: describes a user/group connection to an event series (timestamps, connection `type`/`state`, and `seriesId`).
  - `GetEventSeriesDto`: request body for batch series lookup by IDs.

## [2.2.0] - 2026-03-02

### Added

- **Event time range filters**
  - `GET /event`: optional `from` / `to` query params to filter events by time window (`from` = events ending after this instant, `to` = events starting before this instant).
  - `GET /user/{userId}/events`: optional `from` / `to` range filters with the same semantics; `from` defaults to “now” when omitted.
  - `GET /group/{groupId}/events`: optional `from` / `to` range filters with the same semantics.
  - `GET /events/search`: optional `from` / `to` range filters with the same semantics.
  - `GET /event-series/{seriesId}/events`: optional `from` / `to` range filters with the same semantics.

### Changed

- **Event series instances**: `GET /event-series/{seriesId}/events` summary/description now clarify that the endpoint returns instances ordered by start time and **defaults to only ongoing/upcoming instances** unless a custom `from` / `to` window is provided.

## [2.1.0] - 2026-03-02

### Added

- **Event series API**
  - `POST /event-series`: create an event series from an RRULE and pre-generate all event instances. The creating user (or associated group) becomes the OWNER.
  - `GET /event-series/{seriesId}`: fetch an event series definition.
  - `PUT /event-series/{seriesId}`: update a series and propagate metadata changes to upcoming instances (per-instance overrides are preserved).
  - `DELETE /event-series/{seriesId}`: delete a series and all upcoming event instances; past instances are preserved.
  - `GET /event-series/{seriesId}/events`: list all event instances in a series, ordered by start time.
- **New schemas**: `CreateEventSeriesDto`, `EventSeriesDto`, and `UpdateEventSeriesDto` for creating, returning, and updating event series.
- **Event instances**: `EventDto` now includes an optional `seriesId` linking an instance back to its series.

### Changed

- **Event updates**: `PUT /event/{eventId}` summary clarifies how updates behave for instances in a series—fields supplied in the request body are treated as per-instance overrides and no longer follow series-level updates.
- **Validation**
  - `UpdateUserDto`: `displayName` is no longer required; only `gender` and `birthDate` remain required.
  - `CreateEventDto`: schema no longer requires `displayName`, `startTime`, `endTime`, or `type` (server-side validation may still enforce business rules).
  - Group create/update DTOs that previously required `displayName` now only require `type`.

## [2.0.1] - 2026-02-04

### Changed

#### Connection state enums

- **User connections** (`GET /user/{userId}/connections` query filter and connection DTOs): state enum `requested` | `active` | `rejected` | `blocked` → `incoming_request` | `outgoing_request` | `active`.
- **Event connections** (`GET /event/{eventId}/connections` query filter and connection DTOs): state enum `requested` | `active` | `rejected` | `blocked` → `requested` | `active` | `invited`.
- **Group connections** (connection DTOs): state enum aligned with the same values as event connections: `incoming_request` | `outgoing_request` | `active` | `requested` | `invited`.

#### Connection DTOs (User, Event, Group)

- **EntityConnectionDto** and related connection response DTOs: `state` is no longer in `required`; only `type` is required.
- **type** enum: removed duplicate `admin` and `owner` values from connection type enums.

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
