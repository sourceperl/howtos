# SSH keys memo (shell command memo)

## Keys

### Algorithms in use

- Ed25519 (e.g., `ssh-ed25519`): Modern, elliptic-curve based, fast, highly secure, resistant to side-channel attacks, 
recommended. An Ed25519 key is only 256 bits in size, yet its cryptographic strength is comparable to a 4096-bit RSA 
key.

- RSA (e.g., `ssh-rsa`): Widely used, relies on large prime numbers, strong with 2048+ bit keys.

- DSA (e.g., `ssh-dss`): Older, fixed 1024-bit key size, now considered weak and deprecated.

### Client side

```bash
# create an ed25519 key
ssh-keygen -t ed25519 -C "a comment for my key"
```

build :
- `~/.ssh/id_ed25519` (private key)
- `~/.ssh/id_ed25519.pub` (public key)

## Fingerprints

An SSH fingerprint is a concise, unique cryptographic hash of an SSH server's public key. When you connect to a server 
for the first time, your SSH client displays this fingerprint, prompting you to verify it. By comparing this displayed 
hash with a trusted, out-of-band obtained fingerprint (e.g., from the server administrator), you confirm the server's 
genuine identity. This crucial step prevents "man-in-the-middle" attacks, where a malicious actor might impersonate the 
server to intercept your sensitive data. Your client stores the verified fingerprint in `~/.ssh/known_hosts` for future 
automatic authentication.

### Server side

- View fingerprints

```bash
# lists all keys fingerprints
for f in /etc/ssh/ssh_host_*_key.pub; do ssh-keygen -lf "$f"; done
```

### Client side

- Fetch fingerprints

```bash
# fetch the fingerprints of all keys and append it to your known_hosts file
ssh-keyscan 192.168.1.100 >> ~/.ssh/known_hosts
# view result
ssh-keygen -F 192.168.1.100 | ssh-keygen -lf -
```

- Remove fingerprints

```bash
# remove (some example of remove figerprint of a server)
ssh-keygen -R myserver.example.com
# or for an IP address
ssh-keygen -R 192.168.1.100
# or for an IP address with non-standard port
ssh-keygen -R "192.168.1.100:2222"
```
