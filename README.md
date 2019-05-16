# pks-id-token-extension
allows to extend the expiration time of id-token beyond default 12Hrs

## Background
as of Enterprise PKS 1.4, when configured with OIDC (Active Directory Integration) users are required \
to re-authenticate to Kubernetes every 12 hours. this is a result of Enterprise PKS configuraiton \
that does not configure a value for accessTokenValiditySeconds, and leaves it to the default.

## More information
UAA Configuration https://docs.cloudfoundry.org/api/uaa/version/4.7.0/propMappings/

## Configuration
through BOSH trickery we can configure a new value for jwt configuraiton in PKS manifest \
here are the steps :

1. get the PKS deployment id through BOSH
```
bosh deployments
```
assuming that the result is <b>pivotal-container-service-[hash]</b> i.e. pivotal-container-service-bf45f9e2177d5da24998

alternativly, using jq 
```
bosh deployments --json | jq -r '.Tables[0].Rows[] | select(.name | contains("pivotal-container-service")).name'
```

2. save the current manifest of deployment to filesystem
```
bosh -d pivotal-container-service-bf45f9e2177d5da24998 manifest > pks-manifest.yaml
```

3. Edit the manifest to add the setting 'accessTokenValiditySeconds' with the needed value under jwt policy section
like the following example :
```
        jwt:
          policy:
            accessTokenValiditySeconds: 7776000
            active_key_id: key-1
            keys:
              key-1:
                signingKey: ((uaa_jwt_signing_key_1.private_key))
```

4. apply the new manifest to the deployment
assuming it is called pks-manifest.yaml
```
bosh -d pivotal-container-service-bf45f9e2177d5da24998 deploy pks-manifest.yaml
```

## Issues
```diff
- this is a temporary solution, the next time you run 'Apply Changes' through Ops Manager it will be overriden with the default manifest.
```
