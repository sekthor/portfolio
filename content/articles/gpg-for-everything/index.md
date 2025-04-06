+++
title = 'GPG for everything'
date = 2025-04-05T23:11:48+02:00
tags = ["gpg", "pki"]
technologies = ["gpg"]
+++

GPG (or PGP in general) is awesome.
But it can do a lot more than just encrypting messages.
You can use your GPG key for encryption (confidentiality), for digital signatures (integrity: non-repudiation) and authentication (integrity).
The latter means, we can also use our GPG identity to authenticate SSH connections (rather than using a completely separate SSH key).

To do this efficiently, we can generate a GPG root key and generate subkeys for each of these different tasks.
The neat thing about that is, that we have a single key id (identity) and use this identity for everything.
You might think that this is somewhat of a security risk because we are not keeping a clear separation of responsibilities.
But that is not the case.
We are using a separate subkey for each of those use-cases.
Our root key, the key that ties everything together, we are not actually using for any of those.
In fact, when done properly, we generate our root key pair in a way, that the only capability they can be used for, is to certify our subkeys.

We generate our root key with certify capability only and have it not expire at all or set a long expiration.
Our subkeys we generate for a shorter lifetime.
If we now do this on a secure airgapped system, our holy root key never leaves the secure system.
We only export our subkeys.
We use these for our everyday use until they expire or we revoke them.
If they do, we just generate new subkeys.

## The Plan

1. Generate a new key pair with certify-only capabilities (in my case ECC (curve 22519) without expiration)
1. Generate subkeys for Authentication (SSH), Encryption, and Signatures.
1. Export the sub keys only and transfer them to our everyday system 

## Generating the root key

I have chosen, to use elliptic curve keys for all of my keys.
They are stronger and generate smaller keys.
They are not supported everywhere, so keep that in mind.
But there is nothing keeping you from adding subkeys with different kinds of algorithms to the same root key.

Since they are slightly harder to manage, GPG requires expert mode to be able to generate ecc keys.
To start our key generation, use the following command

```plaintext
gpg --expert --full-gen-key
```

Choose option `ECC (set your own capabilities)` which should be number 11 to make sure to use ECC and to disable all other capabilities.

```plaintext
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 
```

We are prompted for capabilities. 
By default, `Sign` and `Certify` are enabled.
We want only Certify so we toggle `Sign` off by entering `S`. 

```plaintext
Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Sign Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? 
```

Now we can enter `Q`.
We are then prompted for our desired algorithm.

```plaintext
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 
```

I want curve 25519 so i enter `1`.
For expiry, I choose `0`, because I never want this identity to expire.
I don't mind rotating subkeys, but I want my identity to stay the same.

```plaintext
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
```

Then it is just the usual name, email, comment prompt.
Make sure you choose a good passphrase to protect your key.

Perfect.
We now have our root key.
This test example shows, that the key does not have any capabilities beyond certification (`[C]`)

```plaintext
pub   ed25519 2025-04-05 [C]
      2154E59D9A4F60FB215E26C53C27A7BF78005C98
uid                      tester <test@test.test>
```

## Generating the subkeys

Now that we have our root key, we can edit it to add subkeys.
If we want ECC keys, then we will again need the expert flag.

```plaintext
gpg --expert --edit-key <keyid>
```

This will open a gpg prompt.
Type `addkey` to exactly that.
For signing we can use `10`, or use `11` and set signing capabilities only.

```plaintext
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
```
Again, we are choosing curve 25519.
Since this is a subkey I want it to expire.
I find two years (`2y`) a seinsible key lifespan.

Now we can do the same for our encryption key.
Use `addkey` agian and choose option `12` (or `11` to set it manually).

For our SSH key, we want to set our own capabilities (`11`), disable sign and enable authentication.

When all your subkeys are generated, type `save` to save your edits.
And that's basically it. 

your key should look something like this:

```plaintext
gpg -K --with-subkey-fingerprint 2154E59D9A4F60FB215E26C53C27A7BF78005C98
sec   ed25519 2025-04-05 [C]
      2154E59D9A4F60FB215E26C53C27A7BF78005C98
uid           [ultimate] tester <test@test.test>
ssb   ed25519 2025-04-06 [S] [expires: 2027-04-06]
      56BD920C623DFBAE4E6EA259BF37F863BEFE20C6
ssb   ed25519 2025-04-06 [A] [expires: 2027-04-06]
      CC6016484C81682C9044DE42131036D1BEE12F99
ssb   cv25519 2025-04-06 [E] [expires: 2027-04-06]
      1133FE56205EA6DB0D343FC9C8EF63AFD1BFA223
```

Notice, that every subkey only has a single capability (`[S]`, `[A]`, `[E]`).
Our root key can only certify subkeys (`[C]`).

## Exporting your keys to your everyday System

Assuming you generated this key on a dedicated system away from your actual user machine you will need to export your keys, transfer them to the other computer and import them into your keyring.
However, since our root key is really just the certifier for our actual key, we don't need it on our other system.
We only need it when we need new subkeys.
So for security purposes, we leave the root key in its secure location and only export the subkeys.

```plaintext
gpg --export-secret-subkeys <keyid> > keys.gpg
```

You can also use `--armor` if you prefer ASCII text format.

Then transfer to your system and run 

```plaintext
gpg --import keys.gpg
```

I'll advise, that you get rid of that temprary keys file again

```plaintext
shred -u keys.gpg
```
