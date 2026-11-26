# GCP Service Account Key Troubleshooting

## Problem: Invalid/Truncated Service Account JSON

### Symptoms

The provision fails immediately at the network creation step with:

```
Could not deserialize key data. The data may be in an incorrect format,
it may be encrypted with an unsupported algorithm, or it may be an
unsupported key type (e.g. EC curves with explicit parameters).
Details: ASN.1 parsing error: short data (needed at least 3 additional bytes)
```

Or the bootstrap validation fails with:

```
FAIL: PEM body line-length violation (expected 64 chars per middle line,
≤64 for last). line 3: 62 chars; line 18: 62 chars
```

### Root Cause

The GCP service account JSON downloaded from RHPDS (demo.redhat.com) can contain
a **truncated RSA private key** — typically 3 bytes of key material are missing
from the DER structure, manifesting as 2-character-short lines in the PEM body.
The JSON file is structurally valid (parses as JSON) but the embedded private key
is incomplete and cannot be used by any tool (`gcloud`, Python `cryptography`
library, `google.cloud` Ansible modules).

This is a known RHPDS key generation issue. The key was not truncated during
download — it was generated incorrectly.

### Validation

Always validate the service account JSON before attempting deployment:

```bash
python3 scripts/validate-gcp-key.py ~/agnosticd-v2-secrets/gcp-service-account.json
```

Expected output for a valid key:
```
OK: .../gcp-service-account.json is a valid GCP service-account key.
```

### Workaround

If the downloaded RHPDS JSON is invalid, create a fresh service account in
the Open Environment's GCP project:

**Step 1: Authenticate with your personal Google account**

```bash
gcloud auth login --no-browser
# Follow the URL and paste the response
```

**Step 2: Verify you have owner access to the Open Environment project**

```bash
gcloud projects get-iam-policy <PROJECT_ID> \
  --flatten="bindings[].members" \
  --filter="bindings.members:<your-email>" \
  --format="table(bindings.role)"
# Should show roles/owner
```

**Step 3: Create a new service account in your project**

```bash
gcloud iam service-accounts create rhoai-deployer \
    --display-name="RHOAI Deployer" \
    --project=<PROJECT_ID>
```

**Step 4: Grant it owner role**

```bash
gcloud projects add-iam-policy-binding <PROJECT_ID> \
    --member="serviceAccount:rhoai-deployer@<PROJECT_ID>.iam.gserviceaccount.com" \
    --role="roles/owner"
```

**Step 5: Create a fresh key**

```bash
gcloud iam service-accounts keys create \
    ~/agnosticd-v2-secrets/gcp-service-account.json \
    --iam-account=rhoai-deployer@<PROJECT_ID>.iam.gserviceaccount.com \
    --project=<PROJECT_ID>
```

**Step 6: Validate the new key**

```bash
python3 scripts/validate-gcp-key.py ~/agnosticd-v2-secrets/gcp-service-account.json
# Must print OK before proceeding
```

**Step 7: Update vars with the new project ID**

If the RHPDS JSON used `rhpds-XXXXXX` as the project ID but the new SA is in
your Open Environment project, update `vars/rhoai-gcp.yml`:

```yaml
gcp_project_id: <YOUR_OPEN_ENVIRONMENT_PROJECT_ID>
```

Sync to `~/agnosticd-v2-vars/`:
```bash
cp vars/rhoai-gcp.yml ~/agnosticd-v2-vars/rhoai-gcp.yml
cp ~/agnosticd-v2-vars/rhoai-gcp.yml ~/agnosticd-v2-vars/openshift-cluster.yml
```

**Step 8: Update secrets file**

In `~/agnosticd-v2-secrets/secrets-<account>.yml`:
```yaml
gcp_project_id: <YOUR_OPEN_ENVIRONMENT_PROJECT_ID>
gcp_credentials_file: /secrets/gcp-service-account.json
```

**Step 9: Run bootstrap and provision**

```bash
./bootstrap.sh --non-interactive   # must show 9/9 PASS
cd ~/agnosticd-v2 && ./bin/agd provision \
    --guid rhoai01gcp \
    --config openshift-cluster \
    --account <account>
```

---

## Notes

- The EE (Execution Environment) mounts `~/agnosticd-v2-secrets/` as `/secrets/`
  inside the container. Set `gcp_credentials_file: /secrets/gcp-service-account.json`
  (not the host path).
- The `validate-gcp-key.py` script is a port of
  [artemis-edge-acm-demo/scripts/validate-gcp-key.py](https://github.com/tosin2013/artemis-edge-acm-demo/blob/main/scripts/validate-gcp-key.py).
- OCP IPI on GCP requires `roles/owner` or specific compute/DNS/storage roles.
  `roles/owner` is acceptable for sandbox/demo environments.
