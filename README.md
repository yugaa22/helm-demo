https://deploy-adapter.jet.prod.aws.jpmchase.net/swagger-ui/index.html#/publish-controller/publishGKPDeploy


the application is outofsync now.
Reason: They have added  the below config to application yaml but application doesn't support that. instead they need to add it for AppProject
https://argo-cd.readthedocs.io/en/latest/user-guide/orphaned-resources/


CREATE_DATE="2025-07-03T13:30:14.268Z"

# Strip milliseconds and 'Z'
CLEAN_DATE=$(echo "$CREATE_DATE" | sed -E 's/\.[0-9]+Z$//')

# Convert to epoch
epoch=$(date -u -d "$CLEAN_DATE" +"%s")

echo "Epoch time: $epoch"
