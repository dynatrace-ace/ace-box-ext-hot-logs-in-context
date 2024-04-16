# ace-box-ext-template

The `ace-box-ext-template` provides a template structure and examples of how to create a custom [ACE-Box](https://github.com/Dynatrace/ace-box) use case.

## Repository structure

It's important that your external use case complies to a specific folder structure. Most importantly, a folder `roles` need to be available at the repository root that includes at least a `my-use-case` (literal, not renamed) folder.

This `roles` folder and all of it's contents are synced with the ACE-Box's Ansible workdir. Ansible is used to provision use cases including external ones. Upon a successful content sync, Ansible tries to use this `my-use-case` folder as an Ansible role.

An Ansible role is expected to have the following structure:

```
roles/
  my-use-case/
    defaults/
      main.yml
    tasks/
      main.yml
    ...
```

For more information, please see the [official Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html).

The `my-use-case` role can itself source other Ansible roles. Such roles can either be provided as part of the external repository or included from the ACE-Box default roles. A list of ACE-Box roles can be found [here](https://github.com/Dynatrace/ace-box#curated-roles). Please also see the `examples_roles` folder for examples.

## Version and compatibility

Please add a note pointing out which versions of the external use case are compatible with the respective ACE-Box versions:

| Release | Verified against ace-box version |
| --- | --- |
| 1.0.0 | v1.15.0 |

## Variable override (optional)

All default variables can be overriden for an external use case. This can happen in two ways:
- A `defaults/main.yml` is created and added to the respective role directory (e.g. `roles/jenkins/defaults/main.yml`). All external role directories and files are synced with the ACE-Box. The external `main.yml` file would therefore override any ACE-Box default `main.yml`. Attention: This requires that all values in the original `main.yml` are also set in the external `main.yml`.
- A `ace-ext.config.yml` file is added to the repo root. All variables within this file will override the variables set by the ACE-Box. E.g. `my_var_override: foobar` will override the default `my_var_override` set by the ACE-Box.

## Jenkins override (optional)

For most tools, configurations can be applied after they are installed. No so much for Jenkins. Jenkins requires all configuration being present the moment it's installed.

In case your use case requires Jenkins configurations (folders, pipelines, jobs, ...), a special variable `include_jenkins_value_file` can be set that allows including additional configuration. The value of this variable must be a path to a valid Jenkins Helm chart [values file](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/values.yaml). An example of this procedure can be found in the [`demo-default` use case](https://github.com/Dynatrace/ace-box/blob/dev/user-skel/ansible/roles/demo-default/tasks/main.yml).

## Dashboard override (optional)

In case use case specific information shall be shown on the dashboard, there's an option to add custom configuration. This way, use case specific credentials, links to tools, previews and links to guides can be added. An example for custom configuration can be found in the [examples](/example_roles/my-use-case/templates/my-use-case-dashboard.yml.j2).

## DTU provisioning

When an ACE-Box with external use case is provisioned by the DTU team, make sure to grant read access to the Github [ace-box-dtu](https://github.com/orgs/dynatrace-ace/teams/ace-box-dtu) team. This allows them to source the use case during their provisioning process.
