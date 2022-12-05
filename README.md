# OTC-Auth
Open Source CLI for the Authorization with the Open Telekom Cloud.

[![MIT License](https://img.shields.io/apm/l/atomic-design-ui.svg?)](hhttps://github.com/iits-consulting/otc-auth/blob/main/LICENSE)
![Build](https://github.com/iits-consulting/otc-auth/workflows/Build/badge.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/iits-consulting/otc-auth)](https://goreportcard.com/report/github.com/iits-consulting/otc-auth)
![CodeQL](https://github.com/iits-consulting/otc-auth/workflows/CodeQL/badge.svg)
![ViewCount](https://views.whatilearened.today/views/github/iits-consulting/otc-auth.svg)

With this CLI you can log in to the OTC through its Identity Access Manager (IAM) or through an external Identity Provider (IdP) in order to get an unscoped token. The allowed protocols for IdP login are SAML and OIDC. When logging in directly with Telekom's IAM it is also possible to use Multi-Factor Authentication (MFA) in the process.

After you have retrieved an unscoped token, you can use it to get a list of the clusters in a project from the Cloud Container Engine (CCE) and also get the remote kube config file and merge with your local file.

This tool can also be used to manage (create) a pair of Access Key/ Secret Key in order to make requests more secure.

## Login
Use the `login` command to retrieve an unscoped token either by logging in directly with the Service Provider or through an IdP. You can see the help page by entering `login --help` or `login -h`. There are three log in options (`iam`, `idp-saml`, and `idp-oidc`) and one of them must be provided.

### Service Provider Login (IAM)
To log in directly with the Open Telekom Cloud's IAM, you will have to supply the domain name you're attempting to log in to (usually starting with "OTC-EU", following the region and a longer identifier), your username and password. 

`login iam --os-username <username> --os-password <password> --os-domain-name <domain_name>`

Alternatively, it is possible to use MFA if that's desired and/or required. In this case both arguments `--os-user-domain-id` and `--totp` are required. The user id can be obtained in the "My Credentials" page on the OTC. 

`login iam --os-username <username> --os-password <password> --os-domain-name <domain_name> --os-user-domain-id <user_domain_id> --totp <6_digit_token>`

The OTP Token is 6-digit long and refreshes every 30 seconds. For more information on MFA please refer to the [OTC's documentation](https://docs.otc.t-systems.com/en-us/usermanual/iam/iam_10_0002.html).

### Identity Provider Login (IdP)
You can log in with an external IdP using either the `saml` or the `oidc` protocols. In both cases you will need to specify the authorization URL, the name of the Identity Provider (as set on the OTC), as well as username and password for the SAML login and client id (and optionally client secret) for the OIDC login flow. 

#### External IdP and SAML
The SAML login flow is SP initiated and requires you to send username and password to the SP. The SP then authorizes you with the configured IdP and returns either an unscoped token or an error, if the user is not allowed to log in.

`login idp-saml --os-username <username> --os-password <password> --idp-name <idp_name> --os-auth-url <authorization_url>`

At the moment, no MFA is supported for this login flow.

#### External IdP and OIDC
The OIDC login flow is user initiated and will open a browser window with the IdP's authorization URL for the user to log in as desired. This flow does support MFA (this requires it to be configured on the IdP). After being successfully authenticated with the IdP, the SP will be contacted with the corresponding credentials and will return either an unscoped token or an error, if the user is not allowed to log in. 

`login idp-oidc --idp-name <idp_name> --os-auth-url <authorization_url> --client-id <client_id> --client-secret <client_secret>`

The argument `--client-id` is required, but the argument `--client-secret` is only needed if configured on the IdP.

## Cloud Container Engine
Use the `cce` command to retrieve a list of available clusters in your project and/or get the remote kube configuration file. You can see the help page by entering `cce --help` or `cce -h`.

To retrieve a list of clusters for a project use the following command: 

`cce list --os-project-name <project_name>`

To retrieve the remote kube configuration file (and merge to your local one) use the following command:

`cce get-kube-config --os-project-name <project_name> --cluster <cluster_name>`

Alternatively you can pass the argument `--days-valid` to set the period of days the configuration will be valid, the default is 7 days.

## Manage Access Key and Secret Key Pair
You can use the OTC-Auth tool to download the AK/SK pair directly from the OTC. It will download the "ak-sk-env.sh" file to the current directory. The file contains four environment variables.

`otc-auth access-token create`

The "ak-sk-env.sh" file must then be sourced before you can start using the environment variables.

## Environment Variables
The OTC-Auth tool also provides environment variables for all the required arguments. For the sake of compatibility, they are aligned with the Open Stack environment variables (starting with OS).

| Environment Variable | Argument              | Short | Description                                   |
|----------------------|-----------------------|:-----:|-----------------------------------------------|
| OS_AUTH_URL          | `--os-auth-url`       |  N/A  |                                               |
| OS_USERNAME          | `--os-username`       |  `u`  | Username (iam or idp)                         |
| OS_PASSWORD          | `--os-password`       |  `p`  | Password (iam or idp)                         |
| OS_DOMAIN_NAME       | `--os-domain-name`    |  `d`  | Domain Name from OTC Tenant                   |
| OS_USER_DOMAIN_ID    | `--os-user-domain-id` |  `i`  | User id from OTC Tenant                       |
| IDP_NAME             | `--idp-name`          |  `i`  | Identity Provider name (as configured on OTC) |
| CLIENT_ID            | `--client-id`         |  `c`  | Client id as configured on the IdP            |
| CLIENT_SECRET        | `--client-secret`     |  `s`  | Client secret as configured on the IdP        |
| OS_PROJECT_NAME      | `--os-project-name`   |  `p`  | Project name on the OTC                       |
| CLUSTER_NAME         | `--cluster`           |  `c`  | Cluster name on the OTC                       |
