From 239e639865f53c2d3ad6e1c226320f481f630e91 Mon Sep 17 00:00:00 2001
From: metal <metal@i0.tf>
Date: Mon, 20 Feb 2023 21:18:18 +0000
Subject: [PATCH 1/4] initial

Signed-off-by: GitHub <noreply@github.com>
---
 .../1676927683781-AddMessagesIndex.ts         | 33 +++++++++++++++++++
 1 file changed, 33 insertions(+)
 create mode 100644 backend/src/migrations/1676927683781-AddMessagesIndex.ts

diff --git a/backend/src/migrations/1676927683781-AddMessagesIndex.ts b/backend/src/migrations/1676927683781-AddMessagesIndex.ts
new file mode 100644
index 00000000..f4f8cfe7
--- /dev/null
+++ b/backend/src/migrations/1676927683781-AddMessagesIndex.ts
@@ -0,0 +1,33 @@
+import { MigrationInterface, QueryRunner, TableIndex } from "typeorm";
+
+export class AddMessagesIndex1676927683781 implements MigrationInterface {
+  public async up(queryRunner: QueryRunner): Promise<void> {
+    await queryRunner.createIndex(
+      "messages",
+      new TableIndex({
+        columnNames: ["guild_id", "user_id"],
+      }),
+    );
+    await queryRunner.createIndex(
+      "messages",
+      new TableIndex({
+        columnNames: ["guild_id", "channel_id"],
+      }),
+    );
+  }
+
+  public async down(queryRunner: QueryRunner): Promise<void> {
+    await queryRunner.dropIndex(
+      "messages",
+      new TableIndex({
+        columnNames: ["guild_id", "user_id"],
+      }),
+    );
+    await queryRunner.dropIndex(
+      "messages",
+      new TableIndex({
+        columnNames: ["guild_id", "channel_id"],
+      }),
+    );
+  }
+}
-- 
2.25.1


From f6157051af387df5ae7d92c80009d498fda69eb6 Mon Sep 17 00:00:00 2001
From: metal <metal@i0.tf>
Date: Sun, 19 Feb 2023 11:14:43 +0000
Subject: [PATCH 2/4] higher clean limits

Signed-off-by: GitHub <noreply@github.com>
---
 backend/src/plugins/Utility/commands/CleanCmd.ts | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

diff --git a/backend/src/plugins/Utility/commands/CleanCmd.ts b/backend/src/plugins/Utility/commands/CleanCmd.ts
index 6abbb481..6bbd8519 100644
--- a/backend/src/plugins/Utility/commands/CleanCmd.ts
+++ b/backend/src/plugins/Utility/commands/CleanCmd.ts
@@ -11,8 +11,8 @@ import { utilityCmd, UtilityPluginType } from "../types";
 import { LogsPlugin } from "../../Logs/LogsPlugin";
 import { humanizeDurationShort } from "../../../humanizeDurationShort";
 
-const MAX_CLEAN_COUNT = 150;
-const MAX_CLEAN_TIME = 1 * DAYS;
+const MAX_CLEAN_COUNT = 1000;
+const MAX_CLEAN_TIME = 14 * DAYS;
 const MAX_CLEAN_API_REQUESTS = 20;
 const CLEAN_COMMAND_DELETE_DELAY = 10 * SECONDS;
 
-- 
2.25.1


From 8d7dbd649ae93edc1cdf37dd24b4467abbe9d65c Mon Sep 17 00:00:00 2001
From: metal <metal@i0.tf>
Date: Sat, 18 Feb 2023 23:19:07 +0000
Subject: [PATCH 3/4] better archives stuff

Signed-off-by: GitHub <noreply@github.com>
---
 backend/src/data/GuildArchives.ts             |  4 +-
 backend/src/data/GuildSavedMessages.ts        | 16 +++++
 backend/src/data/cleanup/messages.ts          |  4 +-
 backend/src/plugins/Cases/CasesPlugin.ts      |  2 +
 .../src/plugins/Cases/functions/createCase.ts | 18 +++++
 backend/src/plugins/Cases/types.ts            |  2 +
 backend/src/plugins/Utility/UtilityPlugin.ts  | 11 +++
 .../plugins/Utility/commands/ArchiveCmd.ts    | 71 +++++++++++++++++++
 backend/src/plugins/Utility/types.ts          |  1 +
 9 files changed, 125 insertions(+), 4 deletions(-)
 create mode 100644 backend/src/plugins/Utility/commands/ArchiveCmd.ts

diff --git a/backend/src/data/GuildArchives.ts b/backend/src/data/GuildArchives.ts
index b9ae4c05..c10a2024 100644
--- a/backend/src/data/GuildArchives.ts
+++ b/backend/src/data/GuildArchives.ts
@@ -14,7 +14,7 @@ import {
 import { SavedMessage } from "./entities/SavedMessage";
 import { decrypt, encrypt } from "../utils/crypt";
 
-const DEFAULT_EXPIRY_DAYS = 30;
+const DEFAULT_EXPIRY_DAYS = 90;
 
 const MESSAGE_ARCHIVE_HEADER_FORMAT = trimLines(`
   Server: {guild.name} ({guild.id})
@@ -117,7 +117,7 @@ export class GuildArchives extends BaseGuildRepository<ArchiveEntry> {
     guild: Guild,
     expiresAt?: moment.Moment,
   ): Promise<string> {
-    if (expiresAt == null) {
+    if (!expiresAt) {
       expiresAt = moment.utc().add(DEFAULT_EXPIRY_DAYS, "days");
     }
 
diff --git a/backend/src/data/GuildSavedMessages.ts b/backend/src/data/GuildSavedMessages.ts
index e6c51e91..1866b5c2 100644
--- a/backend/src/data/GuildSavedMessages.ts
+++ b/backend/src/data/GuildSavedMessages.ts
@@ -172,6 +172,22 @@ export class GuildSavedMessages extends BaseGuildRepository<SavedMessage> {
     return this.processMultipleEntitiesFromDB(results);
   }
 
+  async getUserMessages(userId, limit?: number): Promise<SavedMessage[]> {
+    let query = this.messages
+      .createQueryBuilder()
+      .where("guild_id = :guild_id", { guild_id: this.guildId })
+      .andWhere("user_id = :user_id", { user_id: userId })
+      .orderBy("posted_at", "DESC");
+
+    if (limit != null) {
+      query = query.limit(limit);
+    }
+
+    const results = await query.getMany();
+
+    return this.processMultipleEntitiesFromDB(results);
+  }
+
   async getMultiple(messageIds: string[]): Promise<SavedMessage[]> {
     const results = await this.messages
       .createQueryBuilder()
diff --git a/backend/src/data/cleanup/messages.ts b/backend/src/data/cleanup/messages.ts
index f94b55d3..c571e415 100644
--- a/backend/src/data/cleanup/messages.ts
+++ b/backend/src/data/cleanup/messages.ts
@@ -8,9 +8,9 @@ import { SavedMessage } from "../entities/SavedMessage";
  * How long message edits, deletions, etc. will include the original message content.
  * This is very heavy storage-wise, so keeping it as low as possible is ideal.
  */
-const RETENTION_PERIOD = 1 * DAYS;
+const RETENTION_PERIOD = 30 * DAYS;
 const BOT_MESSAGE_RETENTION_PERIOD = 30 * MINUTES;
-const DELETED_MESSAGE_RETENTION_PERIOD = 5 * MINUTES;
+const DELETED_MESSAGE_RETENTION_PERIOD = RETENTION_PERIOD; // Because of the archive command, we want to keep those, too
 const CLEAN_PER_LOOP = 100;
 
 export async function cleanupMessages(): Promise<number> {
diff --git a/backend/src/plugins/Cases/CasesPlugin.ts b/backend/src/plugins/Cases/CasesPlugin.ts
index 4f011982..ec3404c5 100644
--- a/backend/src/plugins/Cases/CasesPlugin.ts
+++ b/backend/src/plugins/Cases/CasesPlugin.ts
@@ -17,6 +17,7 @@ import { getTotalCasesByMod } from "./functions/getTotalCasesByMod";
 import { postCaseToCaseLogChannel } from "./functions/postToCaseLogChannel";
 import { CaseArgs, CaseNoteArgs, CasesPluginType, ConfigSchema } from "./types";
 import { InternalPosterPlugin } from "../InternalPoster/InternalPosterPlugin";
+import { GuildSavedMessages } from "../../data/GuildSavedMessages";
 
 const defaultOptions = {
   config: {
@@ -84,5 +85,6 @@ export const CasesPlugin = zeppelinGuildPlugin<CasesPluginType>()({
     pluginData.state.logs = new GuildLogs(pluginData.guild.id);
     pluginData.state.archives = GuildArchives.getGuildInstance(pluginData.guild.id);
     pluginData.state.cases = GuildCases.getGuildInstance(pluginData.guild.id);
+    pluginData.state.savedMessages = GuildSavedMessages.getGuildInstance(pluginData.guild.id);
   },
 });
diff --git a/backend/src/plugins/Cases/functions/createCase.ts b/backend/src/plugins/Cases/functions/createCase.ts
index 0a0925d7..c8cea257 100644
--- a/backend/src/plugins/Cases/functions/createCase.ts
+++ b/backend/src/plugins/Cases/functions/createCase.ts
@@ -5,8 +5,11 @@ import { resolveUser } from "../../../utils";
 import { CaseArgs, CasesPluginType } from "../types";
 import { createCaseNote } from "./createCaseNote";
 import { postCaseToCaseLogChannel } from "./postToCaseLogChannel";
+import { getBaseUrl } from "../../../pluginUtils";
+import { CaseTypes } from "../../../data/CaseTypes";
 
 export async function createCase(pluginData: GuildPluginData<CasesPluginType>, args: CaseArgs) {
+  const casesTypesWithoutArchive = [CaseTypes.Note, CaseTypes.Unban];
   const user = await resolveUser(pluginData.client, args.userId);
   const userName = user.tag;
 
@@ -40,6 +43,21 @@ export async function createCase(pluginData: GuildPluginData<CasesPluginType>, a
     pp_name: ppName,
     is_hidden: Boolean(args.hide),
   });
+  if (!casesTypesWithoutArchive.includes(args.type)) {
+    const messagesToArchive = Array.from(await pluginData.state.savedMessages.getUserMessages(user.id, 50)).sort(
+      (a, b) => (a.posted_at > b.posted_at ? 1 : -1),
+    );
+
+    const archiveId = await pluginData.state.archives.createFromSavedMessages(messagesToArchive, pluginData.guild);
+    const baseUrl = getBaseUrl(pluginData);
+    await createCaseNote(pluginData, {
+      caseId: createdCase.id,
+      modId: mod.id,
+      body: `Automatically archived messages: ${pluginData.state.archives.getUrl(baseUrl, archiveId)}`,
+      automatic: args.automatic,
+      postInCaseLogOverride: false,
+    });
+  }
 
   if (args.reason || args.noteDetails?.length) {
     await createCaseNote(pluginData, {
diff --git a/backend/src/plugins/Cases/types.ts b/backend/src/plugins/Cases/types.ts
index 9ec1a371..6d560961 100644
--- a/backend/src/plugins/Cases/types.ts
+++ b/backend/src/plugins/Cases/types.ts
@@ -6,6 +6,7 @@ import { GuildCases } from "../../data/GuildCases";
 import { GuildLogs } from "../../data/GuildLogs";
 import { tDelayString, tNullable, tPartialDictionary } from "../../utils";
 import { tColor } from "../../utils/tColor";
+import { GuildSavedMessages } from "../../data/GuildSavedMessages";
 
 export const ConfigSchema = t.type({
   log_automatic_actions: t.boolean,
@@ -23,6 +24,7 @@ export interface CasesPluginType extends BasePluginType {
     logs: GuildLogs;
     cases: GuildCases;
     archives: GuildArchives;
+    savedMessages: GuildSavedMessages;
   };
 }
 
diff --git a/backend/src/plugins/Utility/UtilityPlugin.ts b/backend/src/plugins/Utility/UtilityPlugin.ts
index 5a3e6636..1dbff7c4 100644
--- a/backend/src/plugins/Utility/UtilityPlugin.ts
+++ b/backend/src/plugins/Utility/UtilityPlugin.ts
@@ -15,6 +15,7 @@ import { AvatarCmd } from "./commands/AvatarCmd";
 import { BanSearchCmd } from "./commands/BanSearchCmd";
 import { ChannelInfoCmd } from "./commands/ChannelInfoCmd";
 import { CleanArgs, cleanCmd, CleanCmd } from "./commands/CleanCmd";
+import { ArchiveArgs, archiveCmd, ArchiveCmd } from "./commands/ArchiveCmd";
 import { ContextCmd } from "./commands/ContextCmd";
 import { EmojiInfoCmd } from "./commands/EmojiInfoCmd";
 import { HelpCmd } from "./commands/HelpCmd";
@@ -51,6 +52,7 @@ const defaultOptions: PluginOptions<UtilityPluginType> = {
     can_search: false,
     can_clean: false,
     can_info: false,
+    can_archive: false,
     can_server: false,
     can_inviteinfo: false,
     can_channelinfo: false,
@@ -79,6 +81,7 @@ const defaultOptions: PluginOptions<UtilityPluginType> = {
       level: ">=50",
       config: {
         can_roles: true,
+        can_archive: true,
         can_level: true,
         can_search: true,
         can_clean: true,
@@ -106,6 +109,7 @@ const defaultOptions: PluginOptions<UtilityPluginType> = {
       config: {
         can_reload_guild: true,
         can_ping: true,
+        can_archive: true,
         can_about: true,
       },
     },
@@ -145,6 +149,7 @@ export const UtilityPlugin = zeppelinGuildPlugin<UtilityPluginType>()({
     JumboCmd,
     AvatarCmd,
     CleanCmd,
+    ArchiveCmd,
     InviteInfoCmd,
     ChannelInfoCmd,
     MessageInfoCmd,
@@ -167,6 +172,12 @@ export const UtilityPlugin = zeppelinGuildPlugin<UtilityPluginType>()({
       };
     },
 
+    archive(pluginData) {
+      return (args: ArchiveArgs, msg) => {
+        archiveCmd(pluginData, args, msg);
+      };
+    },
+
     userInfo(pluginData) {
       return (userId: Snowflake, requestMemberId?: Snowflake) => {
         return getUserInfoEmbed(pluginData, userId, false, requestMemberId);
diff --git a/backend/src/plugins/Utility/commands/ArchiveCmd.ts b/backend/src/plugins/Utility/commands/ArchiveCmd.ts
new file mode 100644
index 00000000..b77f708b
--- /dev/null
+++ b/backend/src/plugins/Utility/commands/ArchiveCmd.ts
@@ -0,0 +1,71 @@
+import { GuildPluginData } from "knub";
+import { commandTypeHelpers as ct } from "../../../commandTypes";
+import { SavedMessage } from "../../../data/entities/SavedMessage";
+import { getBaseUrl, sendErrorMessage, sendSuccessMessage } from "../../../pluginUtils";
+import { utilityCmd, UtilityPluginType } from "../types";
+
+const DEFAULT_COUNT = 50;
+const MAX_COUNT = 1000;
+
+export async function archiveMessages(
+  pluginData: GuildPluginData<UtilityPluginType>,
+  messagesToArchive: SavedMessage[],
+) {
+  messagesToArchive = Array.from(messagesToArchive).sort((a, b) => (a.posted_at > b.posted_at ? 1 : -1));
+  const archiveId = await pluginData.state.archives.createFromSavedMessages(messagesToArchive, pluginData.guild);
+  const baseUrl = getBaseUrl(pluginData);
+
+  return pluginData.state.archives.getUrl(baseUrl, archiveId);
+}
+
+const opts = {
+  count: ct.number({ option: true, shortcut: "c" }),
+};
+
+export interface ArchiveArgs {
+  userId: string;
+  count?: number;
+}
+
+export async function archiveCmd(pluginData: GuildPluginData<UtilityPluginType>, args: ArchiveArgs | any, msg) {
+  args.count = args.count ?? DEFAULT_COUNT;
+
+  if (args.count > MAX_COUNT || args.count <= 0) {
+    sendErrorMessage(pluginData, msg.channel, `Archive count must be between 1 and ${MAX_COUNT}`);
+    return;
+  }
+
+  const archivingMessage = msg.channel.send("Archiving...");
+  const messagesToArchive = await pluginData.state.savedMessages.getUserMessages(args.userId, args.count);
+
+  if (messagesToArchive.length > 0) {
+    const archiveResult = await archiveMessages(pluginData, messagesToArchive);
+    let responseText = `Archived ${messagesToArchive.length} message${messagesToArchive.length === 1 ? "" : "s"}`;
+
+    responseText += `\n${archiveResult}`;
+    await sendSuccessMessage(pluginData, msg.channel, responseText);
+  } else {
+    const responseText = `Found no messages to archive!`;
+    await sendErrorMessage(pluginData, msg.channel, responseText);
+  }
+
+  await (await archivingMessage).delete();
+}
+
+export const ArchiveCmd = utilityCmd({
+  trigger: ["archive"],
+  description: "Archive a number of messages for a user",
+  usage: "!archive 106391128718245888",
+  permission: "can_archive",
+
+  signature: [
+    {
+      userId: ct.userId(),
+      ...opts,
+    },
+  ],
+
+  async run({ message: msg, args, pluginData }) {
+    archiveCmd(pluginData, args, msg);
+  },
+});
diff --git a/backend/src/plugins/Utility/types.ts b/backend/src/plugins/Utility/types.ts
index 15f6e925..c41959de 100644
--- a/backend/src/plugins/Utility/types.ts
+++ b/backend/src/plugins/Utility/types.ts
@@ -12,6 +12,7 @@ export const ConfigSchema = t.type({
   can_level: t.boolean,
   can_search: t.boolean,
   can_clean: t.boolean,
+  can_archive: t.boolean,
   can_info: t.boolean,
   can_server: t.boolean,
   can_inviteinfo: t.boolean,
-- 
2.25.1


From c893a100f33a10ae8a73e70ec543ffe185a25c5e Mon Sep 17 00:00:00 2001
From: metal <metal@i0.tf>
Date: Sun, 19 Feb 2023 00:43:20 +0000
Subject: [PATCH 4/4] add more case types to not auto-archive

Signed-off-by: GitHub <noreply@github.com>
---
 backend/src/plugins/Cases/functions/createCase.ts | 8 +++++++-
 1 file changed, 7 insertions(+), 1 deletion(-)

diff --git a/backend/src/plugins/Cases/functions/createCase.ts b/backend/src/plugins/Cases/functions/createCase.ts
index c8cea257..a020c09d 100644
--- a/backend/src/plugins/Cases/functions/createCase.ts
+++ b/backend/src/plugins/Cases/functions/createCase.ts
@@ -9,7 +9,13 @@ import { getBaseUrl } from "../../../pluginUtils";
 import { CaseTypes } from "../../../data/CaseTypes";
 
 export async function createCase(pluginData: GuildPluginData<CasesPluginType>, args: CaseArgs) {
-  const casesTypesWithoutArchive = [CaseTypes.Note, CaseTypes.Unban];
+  const casesTypesWithoutArchive = [
+    CaseTypes.Note,
+    CaseTypes.Unban,
+    CaseTypes.Unmute,
+    CaseTypes.Unban,
+    CaseTypes.Deleted,
+  ];
   const user = await resolveUser(pluginData.client, args.userId);
   const userName = user.tag;
 
-- 
2.25.1

