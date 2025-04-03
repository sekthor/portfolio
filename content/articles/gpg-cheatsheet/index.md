+++
title = 'GPG Cheatsheet'
date = 2025-04-02T20:00:00+02:00
tags = ["gpg"]
technologies = ["gpg"]
+++

GPG implements the PGP cryptography standard.
It is an asymmetric encryption standard, also known as "public key" cryptography.

## How it works

Every user generates a pair of keys with a special mathematical relationship.
One key may encrypt a message, which can only be decrypted with the other key.

By keeping one of them absolutely secret (private key) and sharing the other with everyone, people can encrypt messages with your public key and only you (the "recipient") can decrypt them.
Not even the original sending party can undo this encryption.
To send encrypted messages to other people, you obtain their public key, encrypt your message and only the intended recipient will be able to read it.

### Why is this so cool?

We use encryption to send a message, when two conditions apply

1. The message must only be read by the intended recipient and no one else
1. We do not trust the chanel through which we transmit the message

In short we assume, that the message may be accessed in transit, but we want to prevent this third party to be able to read it.
This problem is ancient.
An throughout history we have come up with increasingly more sophisticated ciphers to solve this problem
But up until the mid 1970s, these methods have always be symmetric.
What does that mean?
Encryption always requires some kind of secret unknown to the malicious third party.
In symmetric encryption, this secret is shared among the two communicating parties.
This secret must be shared at some point before the encryption.
And herein lies the problem.
Because, as previously established, we do not trust the chanel we wish to communicate over.
So at some point we need a secure, trusted channel to share our secret (e.g. a passphrase).

This is, what makes asymmetric encryption so fantastic.
The key we encrypt the message with is useless to decrypt the message again.
It can only be done with it's secret counterpart.
The public key we can share with anyone over any chanel and we do not care if the malicious third party get a hold of it.
Thus we never require a secret chanel.
And so long we keep that secret everyone willing can encrypt messages and share those in plain sight if they so choose.
Only the holder of the corresponding private key will ever be able to undo the encryption and read the original message.

## Generating a key pair

To generate a key using all the default configurations, use:

```plaintext {linenos=false}
gpg --gen-key
```
This will prompt you for your name and email and let you set a passphrase to protect your private key with.
Upon completion the output looks something like this:

```plaintext {linenos=false}
pub   rsa3072 2025-04-02 [SC] [expires: 2027-04-02]
      F62BCC799423A19971828986ADB18DF42CB9E6CB
uid                      Your name <your@email.com>
sub   rsa3072 2025-04-02 [E] [expires: 2027-04-02]
``` 

This is a fast and easy way to get yourself a key pair.
Your name can be any string, that is meant to identify you, to a user, that is using your public key.
Similarly, the e-mail is used to associate the key to you (your e-mail address).
You can actually identify keys via the e-mail, however if you have multiple key pairs associated with the same e-mail address this can lead to confusion.
For example, exporting a key by referencing the e-mail, will export **all** keys connected to that email.
So, to be explicit when working with gpg keys, you should always reference them with their key id.
In the above example our key was generated with the key id of `F62BCC799423A19971828986ADB18DF42CB9E6CB`.

### Generating keys with more control

To have a little more control over the parameters of your key (such as the algorithm used), use the `--full-gen-key`.

```plaintext {linenos=false}
gpg --full-gen-key
``` 

This prompts us for a little more options.
However, we still do not have the full range of control we can have.
A (somewhat less) common use-case would be to generate a key pair using elliptic curve cryptography algorithms, rather than your standard RSA or DSA keys.
This does come with some minor pitfalls in key management.
Thus GPG requires us to use the `--expert` flag, to achieve this, so we declare that we know what we're on about.  

```plaintext {linenos=false}
gpg --expert --full-gen-key
```

## Sharing your key

You can now export your public key and share it with the world.
You can choose a binary format (default) or an base64 encoded text-format and save it to a file.

```plaintext {linenos=false}
gpg --export <keyid> > mykey.pub         # binary
gpg --armor --export <keyid> > mykey.pub # text
```

## Importing some one else's public key

To send encrypted messages to someone, they will need to share their public key with you.
You can import this file into your keyring, and then use it to encrypt messages and verify signatures.

```plaintext {linenos=false}
gpg --import <keyfile>
```

## Listing keys

Listing all the public keys in your keyring:

```plaintext {linenos=false}
gpg -k
```

Only listing keys for which you own the private keys:

```plaintext {linenos=false}
gpg -K
```

## Encrypting a file

You encrypt a file for a recipient.
You specify that recipient with the `-r` flag.
Use the `-e` for encryption.

```plaintext {linenos=false}
gpg -e -r <keyid> <file-to-encrypt>
```

This generates an encrypted copy of the file with the `.gpg` file extension.

You can also encrypt a file for multiple recipients:

```plaintext {linenos=false}
gpg -e -r <keyid> -r <keyid2> <file-to-encrypt>
```

You can also pipe input into gpg to encrypt it.
It will then output the encrypted content to `stdout`.
Thus you will have to pipe it into a file.

```plaintext {linenos=false}
cat file-to-encrypt | gpg -e -r <keyid> > file.gpg
```

If you would like to send your messages in a text format instead of binary, you can use the `--armor` (`-a`) flag.

```plaintext {linenos=false}
gpg -e -r <keyid>  --armor <file-to-encrypt>
```

### Encrypting a file with passphrase only

Sometimes we may want to encrypt a file only with a passphrase, as we do not know the public keys of the recipients.

This is obviously not the recommended way of doing this, but can be done as follows:

```plaintext {linenos=false}
gpg -c <filename>
```

## Decrypting a message

Decrypt a file using:

```plaintext {linenos=false}
gpg -d <encrypted-file>
```

Provided you have the private key of any of the recipients in the message, you will be able to decrypt the message.

## Signing a Message

We can actually also use the private key for encryption.
This doesn't make much sense for encryption it self, as anyone possessing the public key can read it.
But since the private key is never shared with anyone and messages can only be decrypted by the corresponding public key, we can assume, that messages that decrypt with it were actually encrypted by that person.
We can validate, that the message does actually originate from the person with that private key and not somebody else.
We call this "signing".

```plaintext {linenos=false}
gpg --output <signedfile>.gpg -s <file-to-sign>
```

If the output flag is omitted it will create a file with the same name plus `.gpg` extension.

If you have multiple keys you may want to specify the key to sign the message with.
Use the `--local-user` flag with the desired key id.

```plaintext {linenos=false}
gpg --local-user <keyid> --output <signedfile>.gpg -s <file-to-sign>
```

### Detaching signature

We have our file `hello.txt` and our singed version of the file `hello.txt.gpg`.
The signed version actually contains a full copy of the file.
So sending the file like that is fine.

However, if you send your `hello.txt` and just want to attach a signature you are sending the file twice.
Not only is this inefficient, the verification is only performed on the file included in the gpg version.

The verify process will actually warn us about this, if it detects `hello.txt` while validating `hello.txt.gpg`.

```plaintext {linenos=false}
gpg: WARNING: not a detached signature; file 'hello.txt' was NOT verified!
```

If you do it this way, sending a file and it's signature separately, you need to "detach" the signature.

The flag for this is `--detach-sign` or `-b` for short.

```plaintext {linenos=false}
gpg gpg -s -a -b <file> 
```

But this also means, that you can not verify the signature without the original file present.
This would cause an error.

```plaintext {linenos=false}
gpg: no signed data
gpg: can't hash datafile: No data
```

## Verifying signatures

To verify a signature run:

```plaintext {linenos=false}
gpg --verify <signed-file>
```

## Putting it all together

To encrypt a message for someone, sign it, encode it to an ASCII format use:

```plaintext {linenos=false}
gpg -sea -r <keyid> <file>
```

- `-e`, `--encrypt`:  encrypt the message
- `-s`, `--sign`: sign the message
- `-a`, `--armor`: use base64 ASCII format
- `-r`, `--recipient`: specify recipient, (multiple possible with multiple `-r` flags)

One thing to note when running sign, and encrypt together, is that the file is signed first and then encrypted.
So you can only verify the signature, if you can also decrypt the message

If you need the signature to be verifiable by everyone you would need to encrypt first and then sign.

```plaintext {linenos=false}
gpg -ea <file>
gpg -sba <file>.asc
```
