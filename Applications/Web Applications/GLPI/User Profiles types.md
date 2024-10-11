---
created: 2023-12-21T16:39
updated: 2024-03-11T19:36
tags:
  - GLPI
  - WebHosting
---
By default, 7 profiles are pre-registered in GLPI.

| Profile      | Description                                                                                                                                                                                                                                                                               |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Super-Admin  | This profile is granted all permissions! **Warning:** if the super-admin profile is deleted or if the simplified interface is associated with this profile, access to the GLPI configuration may be permanently lost.                                                                     |
| Admin        | This profile has administration rights for all GLPI. Some restrictions are applied to it at the level of the configuration of rules, entities as well as other items which may alter the behavior of GLPI.                                                                                |
| Supervisor   | This profile incorporates the elements of the Technician profile by adding elements allowing management of a team and its organization (allocation of tickets, etc.).                                                                                                                     |
| Technician   | This profile corresponds to the one used for a maintenance technician, having read access to the inventory and to the help desk in order to process tickets.                                                                                                                              |
| Hotliner     | This profile corresponds to the one that could be given for a hotline service ; it allows to open tickets and follow them but not to be in charge of them as a Technician can be.                                                                                                         |
| Observer     | This profile has read permission to all inventory and management data. In terms of assistance, it can open a ticket or be assigned one, but cannot administer this section (assign a ticket, steal a ticket, etc.).                                                                       |
| Self-Service | This profile is the most limited. It is also the only one to have a different interface, the simplified interface, as opposed to the standard interface. However, it can declare a ticket, add a follow-up, consult the FAQ or reserve asset. This profile is set as the default profile. |
