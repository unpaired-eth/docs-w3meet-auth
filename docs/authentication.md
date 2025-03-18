# 🔐 Authentication Flow

The authentication process follows these steps:

## 1️⃣ Challenge Generation (Canister)
- The **canister** generates a challenge and stores it.
- The mobile device retrieves the **challenge hash**.

#### Code Example - Challenge Generation:
Below is an example of how the **canister** generates a challenge and stores it:

```javascript
const challenges = new StableStorage<string, Challenge>(0);

registerPasskey(user: User): string {
    try {
      const randomBytes = Array.from({ length: 32 }, () =>
        Math.floor(Math.random() * 256)
      );

      const challenge = Buffer.from(randomBytes).toString("base64");

      challenges.insert(user.id, {
        id: user.id,
        value: challenge,
      });

      return challenge;
    } catch (error) {
      return "Error to get challenge";
    }
  }
```

## 2️⃣ Passkey Authentication (Mobile)
- The user authenticates using a **Passkey**.
- If successful, the mobile app:
  - Validates the user’s **Passkey credentials**.
  - Creates a **Dfinity keypair**.
  
#### Code Example - Passkey Authentication:
Below is an example of how the **mobile** create a passkey and validates it:

```javascript
  Passkey.create({
    challenge,
    user: passkey.user,
    rp: passkey.rp,
    pubKeyCredParams: [
      {
        type: 'public-key',
        alg: -7,
      },
      {
        type: 'public-key',
        alg: -257,
      },
    ],
    timeout: 1800000,
    attestation: 'none',
    excludeCredentials: [],
    authenticatorSelection: {
      authenticatorAttachment: 'platform',
      requireResidentKey: true,
      residentKey: 'required',
      userVerification: 'required',
    },
  });
```  

## 3️⃣ Passkey Validation (Canister)
- The **mobile device** sends:
  - The **Passkey payload**.
  - The **Dfinity public key**.

#### Code Example - Generate Dfinity keypair:
Below is an example of how the **mobile** generate dfinity keypair:

```javascript
  export class KeypairProvider {
    private static pubkeys: TPubKeys = {};

    static create(user: TPasskey['user']) {
      const currentPubkey = KeypairProvider.pubkeys[user.id];

      if (currentPubkey) {
        return currentPubkey;
      }

      const base = Ed25519KeyIdentity.generate();
      const pubkey = toHex(base.getPublicKey().toDer());

      KeypairProvider.pubkeys[user.id] = {
        base,
        pubkey,
      };

      return { pubkey, base };
    }

    const passkey = await PasskeyProvider.create(challenge, this.passkey);

    const payload = {
      id: passkey.id,
      rawId: passkey.rawId,
      response: {
        attestationObject: passkey.response.attestationObject,
        clientDataJSON: passkey.response.clientDataJSON,
      },
    };

    const { pubkey, base } = KeypairProvider.create(this.passkey.user);

    const { error, data } = await this.actor.authValidatePasskey(
      pubkey,
      this.passkey.user,
      payload
    );
```  

- The **canister** validates the payload against the challenge.
  - If **invalid**, the process stops here.
  - If **valid**, the canister proceeds to delegation.

#### Code Example - Validate Passkey:
Below is an example of how the **canister** validate passkey:

```javascript
  const AllowedOrigins = ["https://vercel-endpoint.vercel.app"];

  const clientData = Util.base64URLDecode<ClientDataResponse>(
    payload.response.clientDataJSON,
    true
  );

  if (clientData.type !== "webauthn.create") {
    throw new Error("Invalid passkey method");
  }

  if (!AllowedOrigins.includes(clientData.origin)) {
    throw new Error("Origin not allowed");
  }

  const challenge = challenges.get(user.id);

  if (
    challenge &&
    Util.removeSpecialCharacter(challenge.value) !==
      Util.removeSpecialCharacter(clientData.challenge)
  ) {
    return false;
  }

  return true;
```  

## 4️⃣ Delegation Creation (Canister)
- The **canister** generates a **Dfinity Delegation** using the user’s **public key**.

#### Code Example - Delegation Create:
Below is an example of how the **canister** create delegation:

```javascript
  import { fromHex } from "@dfinity/agent";
  import {
    DelegationChain,
    DelegationIdentity,
    Ed25519KeyIdentity,
    Ed25519PublicKey,
  } from "@dfinity/identity";

  export class KeypairProvider {
    static async createDelegationByPubkey(
      user: User,
      pubkey: string
    ): Promise<DelegationIdentity | null> {
      try {
        const expiration = new Date(Date.now() + 9e5);
        const publicKey = Ed25519PublicKey.fromDer(fromHex(pubkey));

        const randomBytes = crypto.getRandomValues(new Uint8Array(32));
        const signIdentity = Ed25519KeyIdentity.fromSecretKey(randomBytes);

        const chain = await DelegationChain.create(
          signIdentity,
          publicKey,
          expiration
        );

        return DelegationIdentity.fromDelegation(signIdentity, chain);
      } catch (error) {
        console.log("Error to create delegation", error);

        return null;
      }
    }
  }
```  

## 5️⃣ Final Association (Mobile)
- The **mobile app** associates the delegation with its **private key**.
- The user gains access using **Dfinity Delegation**.

#### Code Example - Association Delegation:
Below is an example of how the **mobile** associate delegation:

```javascript
  export class KeypairProvider {
    static getDelegationIdentity(
      key: Ed25519KeyIdentity,
      delegationChain: JsonnableDelegationChain
    ) {
      const chain = DelegationChain.fromJSON(delegationChain);

      if (!isDelegationValid(chain)) {
        throw new Error('Invalid  delegation chain');
      }

      return DelegationIdentity.fromDelegation(key, chain);
    }
  }

  const identity = KeypairProvider.getDelegationIdentity(
    base,
    delegationChain
  );

  identity.getDelegation() // Get delegation
  identity.getPrincipal().toString() // Get principal
  identity.getPublicKey().toDer() // Get public key
``` 

### 📌 Key Features:
✅ Decentralized authentication using **Passkeys**.  
✅ Secure delegation with **Dfinity identity**.  
✅ Challenge-based validation to prevent replay attacks.  

For more information, see [Dfinity Delegation](delegation.md).