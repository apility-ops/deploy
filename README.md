# deploy
Action to create, delete and update environments based

## Usage

Unless a token is specified, the action will make its own. If it needs to do this, you must give it permissions to do so
by giving it `id-token` permissions either to the job or step.
```yaml
permissions:
    contents: read
    id-token: write
```

Basic usage is like this.

Assuming the existence of some 

```yaml

steps:
  - name: 'Get site info'
    id: 'siteInfo'
    uses: 'apility-ops/get-site-info@main'
  - name: 'Create environment for deployment'
    uses: apility-ops/deploy@main
    with:
      action: create
      name: ${{ steps.siteInfo.outputs.siteAlias }}-dev
      template: php-74 # Our hosting provides several templates. This is a hypothetical one.
      env: '{"APP_ENV": "production", "APP_DEBUG": "false"}' # BOTH KEYS AND VALUES MUST BE STRINGS
  - name: Simulate some external work
    run: |
      sleep 30
  - name: 'Destroy environment'
    uses: 'apility-ops/deploy@main'
    with:
      action: delete
      name: ${{ steps.siteInfo.outputs.siteAlias }}-dev
```

## Modes/Actions

The action supports three modes.

### Create
```yaml
with:
  action: create
  name: <environment-name>
  template: <environment-template-name>
  env: <string encoded json>
```

When this action is called, it will attempt to create an hosting environment. 
You must supply a name for the environment that is not already in use. 
Most often its `<site-alias>-<branch-name>`.

### Template parameter
You can supply a specific environment type to host the site in by setting the `template` parameter. 
For example `php-<php-verison-number>`. If none is selected, our default value will be chosen.

**This value is recommended to be set manually per repo to ensure updates will not break deployments in the future.**


### Env parameter
If you want to add or override env variables in this environment you can set them manually by using the `env` parameter.
The env paramtere expects a string serialized json object. An easy way/good example is to use a single quoted string, which reduces the amount of escaping necessary

**!!! BOTH KEYS AND VALUES HERE MUST BE STRINGS !!!**
```yaml
with:
  env: '{"APP_ENV": "production", "APP_DEBUG": "false", "NETS": "${{ steps.nets.outputs.credentials }}"}'
```