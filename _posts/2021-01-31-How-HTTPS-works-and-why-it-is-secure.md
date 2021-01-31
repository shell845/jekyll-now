If clients and servers communicate with HTTP, all messages are sent in plain texts. Anyone intercepts the conversation can eavesdrop and retrieve all information (eavesdropping, tampering, pretending).

We can encrypt our messages to make the conversation more secure. There are two types of encryption:

1. Symmetric encryption: use a single key for encryption and decryption 
2. Asymmetric encryption: use a pair of keys (public key and private key) to encrypt and decrypt

HTTPS actually means HTTP + SSL. The protocol uses asymmetric encryption to exchange keys between server and client (for security), and uses symmetric encryption to encrypt messages (for performance). 

**The HTTPS establish flow**

Step 1 Handshake: client and server exchange keys and verify certificate, then generate session key for coming conversation

Step 2 Conversation: client and server communicate with each other with session key.

![HTTPS handshake](https://user-images.githubusercontent.com/4274250/106400414-4a9a7400-63ec-11eb-8f8d-aa66e48383b5.png)

**HTTPS is secure because**

1. Conversations are encrypted by session key so eavesdroppers cannot decrypt. 
2. Prevent man-in-the-middle attack since only client and server have session key.
3. Trusted CA signed certs are trustworthy

**Why private key can decrypt messaged encrypted by public key**

The public key can be known by everyone. However messages encrypted with the public key can only be decrypted in a reasonable amount of time by using the paired private key.

The basic idea is to find 3 numbers (e, d, n) to fulfil `M^(e*d) (mod n) = M`

How to generate public-private key pair (RSA):

1. pick large prime number p and q, calculate n = p * q
2. pick an exponential number e, e < (p - 1)*(q - 1) and e is relatively prime to (p - 1)*(q - 1)
3. Compose public key with e and n
4. Compute d with (e, p, q), compose private key with d and n
