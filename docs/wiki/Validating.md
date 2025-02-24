## Contents

- [Validate a Docker Deployment](#validate-a-docker-deployment)
- [Validate Running Images in Kubernetes](#validate-running-images-in-kubernetes)
- [Sign In to Your Version of SAS Studio](#sign-in-to-your-version-of-sas-studio)
- [Access SAS Environment Manager](#access-sas-environment-manager)
- [Access SAS Logon and SAS Drive](#access-sas-logon-and-sas-drive)
- [Access CAS Server Monitor](#access-cas-server-monitor)
- [Verify Data Access](#verify-data-access)

## Validate a Docker Deployment

To verify that the Docker process is running, run the following command:

**Note:** Enter the command on a single line. Multiple lines are used here to improve readability.

```
docker ps \
--filter name=sas-programming \
--format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}} \t{{.Size}}"
```

Here are typical results:

```
CONTAINER ID   NAMES             IMAGE                                                                            STATUS         SIZE
9436fbaef7d7   sas-programming   ses.sas.download/va-104.0.0-x64_redhat_linux_7-docker/sas-viya-programming:tag   Up 2 hours     3.54MB (virtual 9.14GB)
```

If the image is not running, here are several recovery actions that you can perform:

1. Run the script in interactive mode.

    1. In the launchsas.sh script, comment out the first command by adding a number sign (#) at the beginning of the line. Uncomment the second command by removing the number sign.

    ```
    # Run in detached mode
    #docker run --detach ${run\_args} $IMAGE "$@"

    # For debugging startup, comment out the detached mode command and uncomment the following

    docker run --interactive --tty ${run\_args} $IMAGE "$@"
    ```

    2. Run the script to launch the image.

        `./launchsas.sh`

    3. The software is deployed with enhanced messaging. You might be able to find the error. If you cannot find the error, the output from this command will be useful if you contact SAS Technical Support.

2. Use SAS\_DEBUG to provide more information about entrypoint script execution.

    1. If necessary, install the vi editor so that you can edit and view the scripts that are generated by the launch process.

       `yum install --assumeyes vi`

    2. In the launchsas script, add a new line to enable debugging.

    ```
    ...
    --env CASENV_CASDATADIR=/cas/data
    --env CASENV_CASPERMSTORE=/cas/permstore
    --env SAS_DEBUG=1
    --publish-all
    ...
    ```

    3. Launch the script with the following command.

       `docker run -it --rm --entrypoint /bin/bash $IMAGE`

    4. Using the vi editor, you can view the scripts in the container and work through the errors.

## Validate Running Images in Kubernetes

Here is an example for validating that a single image pod of sas-programming is running in Kubernetes:

`kubectl get -f run/programming.yml`

For a single-container deployment, run the command from the directory where the programming.yml file is located: $PWD/run/programming.yml.

Here are typical results:

```
NAME                                         DATA   AGE
configmap/sas-viya-single-programming-only   5      16m

NAME                                       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/sas-viya-single-programming-only   ClusterIP   None         <none>        80/TCP    16m

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sas-viya-single-programming-only   1/1     1            1           16m
```

Here is an example for validating running images for a `--type full` build. In this example, the default namespace and manifests directory are shown: sas-viya and builds/full/manifests/kubernetes/deployments/, repectively.

`kubectl -n sas-viya get -f builds/full/manifests/kubernetes/deployments/`

**Tips:** 
- For a `--type multiple` build, the default manifests directory is $PWD/builds/multiple/manifests/kubernetes/deployments.
- To find the namespace, locate the related yaml file in the $PWD/builds/full/kubernetes/namespace or $PWD/builds/multiple/kubernetes/namespace directory.

If you do not get the results that you expect, run the following command to enable the DEBUG option to provide more information:

**Note:** Enter the command on a single line. Multiple lines are used here to improve readability.

```
kubectl get configmap poac-config -o yaml | \
sed &'s|poac.SAS\_DEBUG: "0"|poac.SAS\_DEBUG: "1"|' | \
kubectl replace -f -
```

After you run the command, delete the sas-viya-programming pod. Removing the pod forces the creation of a new sas-viya-programming pod. The new pod will provide additional information about the pod start, which can be reviewed by using the kubectl logs command.

## Sign In to Your Version of SAS Studio

Sign in to SAS Studio to ensure that your default version of SAS Studio has been deployed correctly and is working.

The version of SAS Studio that you are using depends on which type of deployment that you performed:

- If you deployed a programming-only environment, then your environment contains SAS Studio 4.4.
- If you deployed a full environment, then your environment contains both SAS Studio 4.4 and SAS Studio 5.1. By default, you will sign into SAS Studio 5.1.

1. Open SAS Studio from a URL with one of the following formats:

    - For SAS Studio 4.4 via Docker run on port 80:

      `https://docker-host:8443/SASStudio`

    - For SAS Studio 4.4 via Kubernetes:

      `https://ingress-path/SASStudio`

    - For SAS Studio 5.1:

      `https://ingress-path/SASStudioV`

1. Sign in using the credentials for your operating system account.

**Notes:**

- To sign out from SAS Studio, click **Sign Out** on the toolbar. Do not use the **Back** button on your web browser.
- Make a note of the correct URL for your environment to share with any other users of your SAS Viya software.

## Access SAS Environment Manager

**Note:** This section is applicable only if you are using a full deployment. Skip this section if you are using a programming-only deployment.

1. Open SAS Environment Manager from a URL with the following format: `https://ingress-path/SASEnvironmentManager`

1. Sign on as one of the SAS Administrators that you created in [User and Group Requirements](System-Requirements#user-and-group-requirements).

## Access SAS Logon and SAS Drive

**Note:** This section is applicable only if you are using a full deployment. Skip this section if you are using a programming-only deployment.

1. To verify SAS Logon, open it using a URL with this format:

    `https://ingress-path/SASLogon`

2. To verify SAS Drive, open it using a URL with this format:

    `https://ingress-path/SASDrive`

## Access CAS Server Monitor

**Note:** This section is applicable only if you are using a programming-only deployment. Skip this section if you are using a full deployment.

1. To verify that CAS Server Monitor has been successfully deployed, access it by opening a web browser and entering the URL in the address field in one of the following formats:

    - For Docker deployment running on port 8443:

      `https://docker-host:8443/cas-shared-default-http`

      Here is an example:

      `https://docker.company.com:8443/cas-shared-default-http`

    - For a Kubernetes deployment:

      `https://ingress-path/cas-tenant-instance-http`

      Here is an example:

      `https://sas-viya.company.com/cas-acme-default-http`

1. Sign on as one of the SAS Administrators that you created in [User and Group Requirements](System-Requirements#user-and-group-requirements).

**Note:** To access CAS Server Monitor, the password must be set for the cas user ID or other administrative account. For more information, see [CAS Administrator Account Requirements](System-Requirements#cas-administrator-account-requirements) and [Addons](Pre-build-Tasks#addons).

## Verify Data Access

1. Go to the [Validating the Deployment](https://go.documentation.sas.com/?docsetId=dplyml0phy0lax&docsetTarget=n18cthgsfyxndyn1imqkbfjisxsv.htm&docsetVersion=3.4) section in the SAS Viya 3.4 for Linux: Deployment Guide.

1. Locate and open the topic that corresponds to the SAS/ACCESS product that you want to validate. For example, if you want to validate access to Greenplum, go to the topic: _Verify SAS/ACCESS Interface to Greenplum_.

1. Review the topic and copy the code within the topic to your clipboard.

1. Go to SAS Studio and [make sure that you are signed in](#sign-in-to-your-version-of-sas-studio).

1. Perform one of the following steps:

    - To verify with SAS Studio 4.4:

      1. Start a new CAS session.
      1. In the navigation pane, open the **Snippets** section.
      1. Select **Snippets** > **SAS Viya Cloud Analytic Services**.
      1. Right-click **New CAS Session** and select **Open**.
      1. Edit and run the SAS code that you copied in step 3. If any of the verification steps for data access return an error, perform the appropriate configuration steps again.

    - To verify with SAS Studio 5.1:

      1. Start a new CAS session.
      1. In the navigation pane, open the **Snippets** section.
      1. Select **Snippets** > **SAS Viya Cloud Analytic Services**.
      1. Right-click **New CAS Session** and select **Open**.
      1. Edit the SAS code that you copied you copied in step 3.
      1. In the toolbar, click <img src="../viya_runningman.png"> to run the new CAS session code. If any of the verification steps for data access return an error, perform the appropriate configuration steps again.

    - Note: When the snippet is added it will display
      ```
      options cashost="<cas server name>" casport=<port number>;
      ```
      - For single image builds, use localhost and 5570 for the "cas server name" and "port number" respectively.
      - For multiple and full deployments, use sas-viya-cas and 5570 for the "cas server name" and "port number" respectively.