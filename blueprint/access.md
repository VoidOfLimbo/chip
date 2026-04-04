## Visibility & Access Control
Content visibility is determined by a stack of access levels. Each content item (page, component, feature) has a visibility level that determines who can access it. The levels are ordered from lowest to highest:
1. `public` — anyone, including unauthenticated users
2. `platform` — any authenticated user in the platform
3. `chads` — users with an active subscription to any Server
4. `followers` — users who follow the content owner (user or Server)
5. `friends` — users who are in contact list of the content owner (different than mutual followers)
6. `members` — users who are members of the Server that owns the content
7. `supporters` — users with an active subscription to the Server that owns the content
8. `moderators` — users with the moderator role on the Server that owns the content
9. `owner` — the owner of the Server that owns the content

On top of this stack, there are also out-of-band access mechanisms:
- `link` — anyone with the invite link and OTP can access; no authentication required;
- `private` — only specific users of the platform explicitly granted access by the owner can access; access is tied to specific user accounts.
- `superowner` — the owner of the entire platform, with access to all content regardless of visibility settings.

The visibility stack is fully ordered — each level supersedes all levels below it. For example, if a page is set to `followers`, then any user who is a follower of the content owner can access it, regardless of whether they are also a member, supporter, or moderator. If a page is set to `members`, then only users who are members of the Server that owns the page can access it, regardless of whether they are also followers or friends.

There is also `only` mode on visibility from `chads` and above, which allows content owners to specify that only users with the exact access level can access the content. For example, if a page is set to `only:supporters`, then only users with an active subscription to the Server can access it, and users with any other access lever including moderators cannot access it. 

The `owner` will always have access to all content of their server regardless of visibility settings, including `only` mode and they can update the visibility settings of any content item. 

The `superowner` will always have access to all servers and all content regardless of visibility settings, including `only` mode. The changes however will need approval from the `owner` of the server to take effect. `superowner` will also have option to bypass the approval process in case of emergencies or critical issues. This is to ensure that the platform can be effectively managed and maintained while still respecting the autonomy of individual server owners.

All the actions related to visibility and access control will be logged for auditing purposes, including changes to visibility settings, access grants and revocations, and any access attempts by users. This is to ensure transparency and accountability in the management of content visibility and access on the platform. Audit logs will be accessible to server owners and the superowner, with appropriate access controls to protect user privacy and security. Any CRUD action on content items will also trigger notifications to the owner and relevant users based on the visibility settings, to keep them informed about changes to the content they have access to.

## Access Tiers
There are basically 3 access tiers on the platform:
1. **Free**: This is the default tier for all users. It includes access to all `public` content and features, as well as the ability to create and manage their own profile, follow other users and servers, and interact with public content.
2. **Premium**: This tier is for users who have paid to have their own server on the platform. These users can subscribe to features to be accessible in their own server. They also have access to invite peoples to the server who can have `Free` tier but can access the content of the server based on the visibility settings. This is to allow multiple users to access the features and utilize the subscription limitations more efficiently. 
3. **Loyalist**: This tier is for users who have an active subscription to the entire platform. It includes access to all content and features, as well as administrative tools and capabilities. This tier is intended for users who want to have the most comprehensive experience on the platform and support its ongoing development and maintenance. Users in this tier will also have the ability to create and manage their own servers, as well as access to exclusive content and features that are not available to users in the Free or Premium tiers. This tier is designed for users who are highly engaged with the platform and want to have the most comprehensive experience possible. They will have access to the insider community and be able to provide feedback and suggestions directly to the development team, as well as receive early access to new features and updates. This tier is intended for users who are passionate about the platform and want to support its growth and success in a meaningful way.

*Note: `superowner` does not belong to any tier and will have highest level of access*