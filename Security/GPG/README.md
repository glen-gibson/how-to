# Beginners Guide to GnuPG

I've found myself interested in using PGP to secure my communication and some of my sensitive data, so here I'll be saving a few tips I've come across along my journeys.  

__GOALS:__
- Create separate keys for certification, encryption and signing
- Store the certification key offline for safe keeping and to protect from compromise
- Distribute my public key so that others can find my public key to encrypt data and communications meant for me

## Table of Contents
- [Key Pair Generation](#key-pair-generation)
  - [Default Key Pair Generation](#default-key-pair-generation)
  - [Full Key Pair Generation](#full-key-pair-generation)
  - [Expert Key Pair Generation](#expert-key-pair-generation) (Recommended)
- [Subkeys](#subkeys)
  - [Finding the Key ID](#finding-the-key-id)
  - [Adding a new Subkey](#adding-a-new-subkey)
- [Hardware Devices](#hardware-devices)
- [To be Continued...](#to-be-continued)
- [References](#references)


</br>

## Key Pair Generation
There are a few ways to generate your key pairs, so I'm going to examine each one.

### Default Key Pair Generation
I've found that most of the documentation I've come across starts off with:
```bash
gpg --generate-key
```

The problem with this command is that it does everything for you.  The end result is sane, but could be a lot better:
```bash
pub   ed25519 2025-08-31 [SC] [expires: 2028-08-30]
      3BD97AE449709A148D9BF862A7AB5B97D984D68B
uid                      Glen Gibson <glen@notmyemailaddress.com>
sub   cv25519 2025-08-31 [E] [expires: 2028-08-30]
```

### Full Key Pair Generation
The next one, asks you a few more questions such as the encryption type you'd like to use and how long the certificates should be valid for (these can be extended later on if needed).  However, it still has limitations.
```bash
gpg --full-generate-key
```
It then prompts for encryption details (I selected ECC and Curve 25519):
```bash
Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection? 9
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at Wed Sep  1 08:02:58 2027 NZST
Is this correct? (y/N) y
```
You're then prompted for your name and email just like the default.

### Expert Key Pair Generation
This is my preference.  It gives you full control over the key generation process and allows you to limit your master key just to certification of subkeys - meaning you can then store the master key offline and only worry about subkey protection and compromise.  A little more work but I definitely recommend it.

```bash
gpg --full-generate-key --expert
```
The `--expert` flag, lets you specifically choose what your master key is responsible for.  You are now offered option 11:
```bash
(11) ECC (set your own capabilities)
```
We're now presented with these options for our master key:
```bash
Possible actions for this ECC key: Sign Certify Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished
```
If we type `S` and press `Enter`, we are now _ONLY_ using the master key for certifying subkeys - nothing else, which is what we want.  We will create separate subkeys for `sign`, `certify` and even `authenticate` if we choose to use it.

The rest of the process is pretty much the same as `--full-generate-key`.  Now we need to create the [subkeys](#subkeys) we'll use for the other functions.

</br>

## Subkeys
Following on from [Expert Key Pair Generation](#expert-key-pair-generation).  

### Finding the Key ID
In order to add subkeys, we need to find our Key ID.
```bash
❯ gpg -k
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2027-08-31
/home/glen/.gnupg/pubring.kbx
-----------------------------
pub   ed25519 2025-08-31 [C] [expires: 2027-08-31]
      2B7511E580088AB67DD99F5C2F348B0E415B546A
uid           [ultimate] Glen Gibson <glen@notmyemailaddress.com>
```
In the case of this one I've created, my Key ID is `2B7511E580088AB67DD99F5C2F348B0E415B546A` so I will run the following to enter edit mode on it:
```bash
gpg --edit-key --expert 2B7511E580088AB67DD99F5C2F348B0E415B546A
```
You will then be presented with the `gpg>` prompt, where you can enter `gpg` specific commands.  If you type `?` you'll get a list of main of the commands available to you.

### Adding a new Subkey
From the `gpg>` prompt we can type `addkey` and press `Enter` to being the subkey creation wizard.
```bash
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
Your selection?
```
Usually you'll want a `sign` key and an `encrypt` key.  The process is the same for both apart from choosing `10` for `sign` and `12` for `encrypt`.  I give ehse a shorter expiry time than the master key.  

>__A word about hardware devices:__  One thing to be aware of, if you're using hardware devices, check their capabilities - some algorithms are not supported depending on the device's firmware capabilities.

The wizard is the same as `--full-generate-key` and once you've created your `sign` and `encrypt` keys you can type `quit` to exit.  You'll then need to confirm you wish to save the changes.

</br>

## Hardware Devices
I have the following devices made by Token2 (I am not affiliated):

- [PIN+ Dual Release3.2 - FIDO2.1 Key with OpenPGP and OTP and Dual USB Ports](https://www.token2.com/shop/product/pin-dual-release3-fido2-1-key-with-openpgp-and-otp-and-dual-usb-ports)
- [T2F2-NFC-Card PIN+ Release3](https://www.token2.com/shop/product/t2f2-nfc-card-pin-release3)

In my case, unless Token2 releases a new firmware with an update for the OpenPGP java applet on the T2F2 card, [I cannot use Curve 25519](https://www.token2.com/site/page/openpgp-setup-guide-for-usb-keys-and-cards) for my key and will need to choose one that is compatible but still secure.  The PIN+ Dual firmware includes an OpenPGP java applet that [does support Curve 25519 (ed25519 / cv25519)](https://www.token2.com/site/page/openpgp-setup-guide-for-usb-keys-and-cards).  Curve 25519 is prefered (where possible) as the key size is smaller while still being more secure than others, and it is fast.

>I do have to say, the T2F2 card is still a very cost effective solution despite the lack of Curve 25519 support and the credit card form factor is quite convenient.  It still supports `RSA2048` which is very common, as well as NIST `P-256` (secp256r1), `P-384` (secp384r1) and `P-521` (secp521r1) elliptic curve algorithms.  It does however require an NFC or SmartCard reader equiped device to be used.

</br>

## To be Continued...
There's still a bit to document as I figure them out.  Off the top of my head:
- Distributing your Public Keys for others to find
- Taking your `certification` master key offline to protect it now that we have subkeys for `sign` and `encrypt` functionality
- Moving private keys to hardware based tokens for extra protection
- Other stuff I haven't thought of....

</br>

## References
Unfortunately, I haven't recorded all the numerous guides I've been through, but my favourite is probably this one:  
https://dev.to/govindup63/gpg-for-noobs-17od

OpenPGP cryptographic algorithms for Token2 devices:  
https://www.token2.com/site/page/openpgp-setup-guide-for-usb-keys-and-cards