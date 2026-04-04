## Visibility & Access Control
Content visibility is determined by a stack of access levels. Each content item (page, component, feature) has a visibility level that determines who can access it. The levels are ordered from lowest to highest:

| Level | Who Can Access |
|---|---|
| `public` | Anyone, including unauthenticated users |
| `platform` | Any authenticated platform user |
| `chads` | Users with an active subscription to any Server |
| `followers` | Users following the content owner (user or Server) |
| `friends` | Users in the content owner's contact list (not the same as mutual followers) |
| `members` | Members of the Server that owns the content |
| `supporters` | Active subscribers to the owning Server |
| `moderators` | Moderators of the owning Server |
| `owner` | The Server owner — always has full access |

On top of this stack, there are also out-of-band access mechanisms:
- `link` — anyone with the invite link and OTP can access; no authentication required;
- `private` — only specific users of the platform explicitly granted access by the owner can access; access is tied to specific user accounts.
- `superowner` — the owner of the entire platform, with access to all content regardless of visibility settings.

The visibility stack is fully ordered — each level supersedes all levels below it. For example, if a page is set to `followers`, then any user who is a follower of the content owner can access it, regardless of whether they are also a member, supporter, or moderator. If a page is set to `members`, then only users who are members of the Server that owns the page can access it, regardless of whether they are also followers or friends.

There is also `only` mode on visibility from `chads` and above, which allows content owners to specify that only users with the exact access level can access the content. For example, if a page is set to `only:supporters`, then only users with an active subscription to the Server can access it, and users with any other access lever including moderators cannot access it. 

The `owner` will always have access to all content of their server regardless of visibility settings, including `only` mode and they can update the visibility settings of any content item. 

The `superowner` will always have access to all servers and all content regardless of visibility settings, including `only` mode. The changes however will need approval from the `owner` of the server to take effect. `superowner` will also have option to bypass the approval process in case of emergencies or critical issues. This is to ensure that the platform can be effectively managed and maintained while still respecting the autonomy of individual server owners.

All the actions related to visibility and access control will be logged for auditing purposes, including changes to visibility settings, access grants and revocations, and any access attempts by users. This is to ensure transparency and accountability in the management of content visibility and access on the platform. Audit logs will be accessible to server owners and the superowner, with appropriate access controls to protect user privacy and security. Any CRUD action on content items will also trigger notifications to the owner and relevant users based on the visibility settings, to keep them informed about changes to the content they have access to.

### Demo Access
- Content owners can mark individual Pages or features as **demoable**.
- Any user can request demo access to a demoable item; the server owner approves or denies the request.
- Approval grants a time-limited access window configured by the server owner (duration, scope).
- Demo sessions are logged and count toward the server's audit trail.
- When a demo period expires, the user's access automatically reverts to their standard visibility level.
- The Super Owner can review and revoke demo access across any server if required.

## Access Tiers
There are 3 access tiers on the platform:

| Tier | How to Get It | Key Capabilities |
|---|---|---|
| **Free** | Default on registration | Public content, profile, follow users & servers |
| **Premium** | One-time Server purchase | Own Server, invite members, configure subscriptions |
| **Loyalist** | Recurring platform subscription | All Free + Premium benefits, enhanced profile, insider access |

### Free
- Default tier for all registered users.
- Access to all `public` content and the Limbo server.
- Can create and manage a profile, follow users and servers, and interact with public content.
- Can be invited as a member of a Premium server and access its content based on visibility settings.

### Premium
- Unlocked by paying a one-time Server creation fee.
- Can create and own a Server, configure pages, set visibility rules, and offer server subscriptions.
- Can invite Free-tier users as server members, allowing them to benefit from the Server's features within the visibility constraints.

### Loyalist
- Active recurring platform subscription.
- Includes all Free and Premium capabilities.
- Access to enhanced profile features (animated media, special effects, custom overlays).
- Insider community access — early feature previews, direct feedback channel to the development team.
- Exclusive platform-wide content and features not available to Free or Premium tiers.

*Note: `superowner` does not belong to any tier and has the highest level of access across the entire platform.*