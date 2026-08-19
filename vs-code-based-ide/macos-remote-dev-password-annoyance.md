# Problem

I constantly be asked for SSH key password each time I change a directory
on my Mac.

# Cause

Your ssh key should be registered with the keychain.

# Fix

Run the following command in your terminal (I am using id_ed25519 yours may be different)

    ssh-add --apple-use-keychain ~/.ssh/id_ed25519

  1. Enter the passphrase for id_ed25519 when prompted.
  2. The passphrase will be saved to your macOS Keychain and loaded automatically across reboots.

  ### Verify

  To verify that the key is loaded in your agent:

    ssh-add -l

# Don't Bother

Don't bother with the ./ssh/config file.
