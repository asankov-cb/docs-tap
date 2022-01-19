# Sample public image scan with compliance check

## <a id="public-image-scan"></a> Public image scan

The following example performs an image scan on an image in a public registry. This image revision has 223 known vulnerabilities (CVEs), spanning a number of severities. ImageScan uses the ScanPolicy to run a compliance check against the CVEs.

The policy in this example is set to only consider `Critical` severity CVEs as a violation, which returns 21 Unknown Severity Vulnerability.

> **Note:** This example ScanPolicy has been deliberately constructed to showcase the features available and should not be considered an acceptable base policy.

In this example, the scan does the following (currently):

* Finds all 223 of the CVEs.
* Ignores any CVEs with severities that are not unknown severities.
* Indicates in the `Status.Conditions` that 21 CVEs have violated policy compliance.

### Define the ScanPolicy and ImageScan

Create `sample-public-image-scan-with-compliance-check.yaml`:

```
---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ScanPolicy
metadata:
  name: sample-scan-policy
spec:
  regoFile: |
    package policies

    default isCompliant = false

    # Accepted Values: "UnknownSeverity", "Critical", "High", "Medium", "Low", "Negligible"
    violatingSeverities := ["Critical"]
    ignoreCVEs := []

    contains(array, elem) = true {
      array[_] = elem
    } else = false { true }

    isSafe(match) {
      fails := contains(violatingSeverities, match.Ratings.Rating[_].Severity)
      not fails
    }

    isSafe(match) {
      ignore := contains(ignoreCVEs, match.Id)
      ignore
    }

    isCompliant = isSafe(input.currentVulnerability)

---
apiVersion: scanning.apps.tanzu.vmware.com/v1beta1
kind: ImageScan
metadata:
  name: sample-public-image-scan-with-compliance-check
spec:
  registry:
    image: "nginx:1.16"
  scanTemplate: public-image-scan-template
  scanPolicy: sample-scan-policy
```

### (Optional) Set up a watch

Before deploying, set up a watch in another terminal to view the process:

```
watch kubectl get scantemplates,scanpolicies,sourcescans,imagescans,pods,jobs
```

For more information about setting up a watch, see [Observing and Troubleshooting](../observing.md).

### Deploy the resources

```
kubectl apply -f sample-public-image-scan-with-compliance-check.yaml
```

### View the scan results

```
kubectl describe imagescan sample-public-image-scan-with-compliance-check
```

Note that the `Status.Conditions` includes a `Reason: EvaluationFailed` and `Message: Policy violated because of 18 CVEs`.

For more information about scan status conditions, see [Viewing and Understanding Scan Status Conditions](../results.md).

### Modify the ScanPolicy

See the previous source scan example.

### Clean up

To clean up, run:

```
kubectl delete -f sample-public-image-scan-with-compliance-check.yaml
```