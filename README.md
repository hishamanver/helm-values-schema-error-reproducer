# helm-values-schema-error-reproducer
Reproduces issue in https://github.com/helm/helm/issues/31570

## Steps

### Step 1 - Download working and broken versions of helm (added to repo as a backup)
Install version 3.19.2 from https://github.com/helm/helm/releases

```
wget https://get.helm.sh/helm-v3.19.2-linux-amd64.tar.gz  # skip if using tar.gz from this repo

mkdir helm-3.19.2
tar -xvf helm-v3.19.2-linux-amd64.tar.gz -C helm-3.19.2

wget https://get.helm.sh/helm-v3.18.4-linux-amd64.tar.gz  # skip if using tar.gz from this repo

mkdir helm-3.18.4
tar -xvf helm-v3.18.4-linux-amd64.tar.gz -C helm-3.18.4
```

### Step 2 - Tests

#### Success

`./helm-3.18.4/linux-amd64/helm lint reproducer-chart`

execution:

```
➜  helm-values-schema-error-reproducer git:(main) ✗ ./helm-3.18.4/linux-amd64/helm lint reproducer-chart
==> Linting reproducer-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

#### Failure

`./helm-3.19.2/linux-amd64/helm lint reproducer-chart`

execution:

```
➜  helm-values-schema-error-reproducer git:(main) ✗ ./helm-3.19.2/linux-amd64/helm lint reproducer-chart
==> Linting reproducer-chart
[INFO] Chart.yaml: icon is recommended
[ERROR] values.yaml: "file:///values.schema.json#" is not valid against metaschema: jsonschema validation failed with 'http://json-schema.org/draft-07/schema#'
- at '/properties/autoscaling/properties/minReplicas/enum': items at 0 and 1 are equal
[ERROR] templates/: values don't meet the specifications of the schema(s) in the following chart(s):
reproducer-chart:
"file:///values.schema.json#" is not valid against metaschema: jsonschema validation failed with 'http://json-schema.org/draft-07/schema#'
- at '/properties/autoscaling/properties/minReplicas/enum': items at 0 and 1 are equal

Error: 1 chart(s) linted, 1 chart(s) failed
```

## Appendix

### reproducer-chart creation steps (must be done after step 1 above)

```
./helm-3.19.2/linux-amd64/helm create reproducer-chart
./helm-3.19.2/linux-amd64/helm lint reproducer-chart # SUCCESS
```

Add below to values.schema.json in the root of the helm chart

```
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "properties": {
    "autoscaling": {
      "description": "autoscaling properties",
      "properties": {
        "minReplicas": {
            "enum": ["1", 1]
        }
      },
      "type": "object"
    }
  },
  "title": "Values",
  "type": "object"
}
```

```
./helm-3.19.2/linux-amd64/helm lint reproducer-chart # FAILS
```
