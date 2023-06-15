# gh-action-rl-scanner-simple

* This action runs a scan with reversinglabs/rl-scanner on a single provided artifact.
* This action expects the artifact to be produced in the current workspace before this action is called
* The path to the artifact to scan relative to the root of the github repo is specified as input to the action.
* optionally you can specify the report directory name

## Environment
The this action requires the licence data to be passed via the environment
with the pre set names as required by the reversinglabs/rl-scanned docker image:

* RLSECURE_SITE_KEY
* RLSECURE_ENCODED_LICENSE

The most secure way is to define secrets on the organisational or repository level.

## Inputs

### `artifact-to-scan`
* **Required** the relative filepath (relative to your github repo)

### `report-path`
* **Optional**: the report directory ( **default**: MyReportDir )

## Outputs

* `description` The status string from rl-scanner, containing FAIL or PASS.

* `status`  The status word as s used by the github status api: one of:
    * error: after scanning someting went wrong: we have no result string.
    * failure: the result string has FAIL.
    * success: the result string has PASS.

## Artifact
* the scan action produces a new report directory in ${{ inputs.report-path }}/report

## Example usage

### Scan
    # run the action
    - name: run the reversinglabs/rl-scanner
      id: scan
      env: # Or as an environment variable
        RLSECURE_ENCODED_LICENSE: ${{ secrets.RLSECURE_ENCODED_LICENSE }}
        RLSECURE_SITE_KEY: ${{ secrets.RLSECURE_SITE_KEY }}
      uses: reversinglabs/gh-action-rl-scanner-simple@v1
      with:
        my-artifact-to-scan: 'README.md'

Will scan the README.md and produce a report in ${{ inputs.report-path }}/report that can be picked up from the workspace.

### Status
As we always produce a status even when the scan step detects a error (and fails),
you should place a explicit if condition as shown so that the status always gets extracted on PASS and FAIL of the scan.

    # -------------------------------------
    # Use the output from the scan step
    - name: Get the scan status output
      if: success() || failure()
      run: |
        echo "The status is: '${{ steps.scan.outputs.status }}'"
        echo "The description is: '${{ steps.scan.outputs.description }}'"

### Report
The report can be extracted as artifact with the upload-artifact action step.

As we always produce a report even when the scan step detects a error (and fails),
you should place a explicit if condition as shown so that the report always gets uploaded on PASS and FAIL of the scan.

    # -------------------------------------
    - name: Archive Report
      if: success() || failure()
      uses: actions/upload-artifact@v3
      with:
        name: report
        path: ${{ inputs.report-path }}

## References
 * [reversinglabs/rl-scanner Docker](https://hub.docker.com/repository/docker/reversinglabs/rl-scanner/)
 * [gh-action-rl-scanner-composite](https://github.com/marketplace/actions/gh-action-rl-scanner-composite)
