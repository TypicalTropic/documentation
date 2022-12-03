---
description: >-
  This code snippet documents how to retrieve access token with next auth using
  t3 stack.
---

# Discord OAuth Access Token With Next Auth

```typescript
// [...nextauth].ts

import NextAuth, { type NextAuthOptions } from "next-auth";
import DiscordProvider from "next-auth/providers/discord";
// Prisma adapter for NextAuth, optional and can be removed
import { PrismaAdapter } from "@next-auth/prisma-adapter";

import { env } from "../../../env/server.mjs";
import { prisma } from "../../../server/db/client";

export const authOptions: NextAuthOptions = {
  callbacks: {
    async jwt({ token, account, profile }) {
      // Persist the OAuth access_token and or the user id to the token right after signin
      if (account) {
        token.accessToken = account.access_token;
        token.id = account.userId;
      }
      return token;
    },
    async session({ session, token, user }) {
      // Add access token from JWT to session for access on client side
      session.accessToken = token.accessToken;
      if (session.user) {
        session.user.id = token.id as string;
      }

      return session;
    },
  },
  // Configure one or more authentication providers
  adapter: PrismaAdapter(prisma),
  providers: [
    DiscordProvider({
      clientId: env.DISCORD_CLIENT_ID,
      clientSecret: env.DISCORD_CLIENT_SECRET,
      // Retrieve token from discord
      token: "https://discord.com/api/oauth2/token"
      // Overide default scope to include guilds
      authorization: { params: { scope: "identify email guilds" } },
    }),
    // ...add more providers here
  ],
  
  // You need to specify these additional settings to enable JWT
  session: {
    strategy: "jwt",
  },
  secret: env.NEXTAUTH_SECRET,
  jwt: {
    secret: env.NEXT_AUTH_JWT_SECRET,
  },
};

export default NextAuth(authOptions);
```

```
// types/next-auth.d.ts

import { type DefaultSession } from "next-auth";

declare module "next-auth" {
  /**
   * Returned by `useSession`, `getSession` and received as a prop on the `SessionProvider` React Context
   */
  interface Session {
    accessToken?: unknown;
    user?: {
      id: string;
    } & DefaultSession["user"];
  }
}

```
