## The Issue
<!-- Resolves #REPLACE_ME_WITH_ISSUE_NUMBER -->
<!-- Briefly describe the issue or feature. -->

## How This PR Solves The Issue
<!-- Describe the key change(s) in this PR that address the issue above. -->

## Manual Testing Instructions
```bash
# Test with this add-on with these steps:
ddev poweroff && ddev delete images -y
ddev config --auto
ddev add-on get https://github.com/mittwald/ddev-mittwald/tarball/REPLACE_ME_WITH_PR_BRANCH_NAME#REPLACE_ME_WITH_PR_NUMBER
ddev restart
# Proceed with testing...
```

## Automated Tests
<!-- What automated tests are there? Or why are automated tests not needed? -->

## Release/Deployment Notes
<!-- Will this require changes to other repositories? -->
<!-- Does this change need to be reflected in the documentation? -->
