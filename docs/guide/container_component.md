# Page Title (Put Your Page Title Here)

<!-- DO NOT SUBMIT without changing the go-link -->

go/g3doc-canonical-go-links

<!--*
# Document freshness: For more information, see go/fresh-source.
freshness: { owner: 'avolkov' reviewed: '2020-06-01' }
*-->

This is a g3doc Markdown page. For more information, see the Markdown Reference
(go/g3doc-reference) and the g3doc Style Guide (go/g3doc-style).

[TOC]

## Tips

*   DO NOT SUBMIT without replacing the sample text by your content.
*   Make sure your page has a page title (see above)
*   Keep this file formatted using go/mdformat.

# Building Container-based components

## Creating container-based components

Container-based components are backed by containerized command-line programs. If
you already have a container, you can use TFX to create a component based on it,
declaring inputs and outputs. The create_container_component function The
`create_container_component` creates a container-based component. Function
parameters: name: The name of the component

inputs: A dictionary that maps input names to types. outputs: A dictionary that
maps output names to types parameters: A dictionary that maps parameter names to
types

image: Container image name. command: Container entrypoint command line. Not
executed within a shell. The command line can use placeholder objects that are
replaced at compilation time with the input, output, or parameter. The
placeholder objects can be imported from
tfx.dsl.component.experimental.placeholders. Note that Jinja templates are not
supported.

Return value: Component (class inheriting from base_component.BaseComponent)
that can be instantiated and used inside the pipeline.

### Placeholders

For a component that has inputs or outputs, the `command` usually needs to have
placeholders that are replaced with actual data at runtime.
InputValuePlaceholder A placeholder for the value of the input argument.
Represents a placeholder that is replaced at runtime with the string value of
the input argument of an execution property. InputUriPlaceholder A placeholder
for the URI of the input artifact argument. Represents a placeholder that is
replaced at runtime with the URI of the input artifact argument data.
OutputUriPlaceholder Represents a placeholder for the URI of the output artifact
argument. Represents a placeholder that is replaced at runtime with the URI for
the output artifact data.

Example: ``` from tfx.dsl.component.experimental import container_component from
tfx.dsl.component.experimental import placeholders from tfx.types import
standard_artifacts

component = container_component.create_container_component( name='TrainModel',
inputs={ 'training_data': standard_artifacts.Dataset, }, outputs={ 'model':
standard_artifacts.Model, }, parameters={ 'num_training_steps': int, },
image='gcr.io/my-project/my-trainer', command=[ 'python3', 'my_trainer',
'--training_data_uri', placeholders.InputUriPlaceholder('training_data'),
'--model_uri', placeholders.OutputUriPlaceholder('model'),
'--num_training-steps',
placeholders.InputValuePlaceholder('num_training_steps'), ] ) ```

An example of a non-python component that downloads, transforms and uploads the
data:

```python
from tfx.dsl.component.experimental import container_component
from tfx.dsl.component.experimental import placeholders
from tfx.types import standard_artifacts

grep_component = container_component.create_container_component(
    name='FilterWithGrep',
    inputs={
        'text': standard_artifacts.ExternalArtifact,
    },
    outputs={
        'filtered_text': standard_artifacts.ExternalArtifact,
    },
    parameters={
        'pattern': str,
    },
    # The component code uses gsutil to upload the data to Google Cloud Storage, so the
    # container image needs to have gsutil installed and configured.
    image='google/cloud-sdk:278.0.0',
    command=[
        'sh', '-exc',
        '''
          pattern="$0"
          text_uri="$1"
          text_path=$(mktemp)
          filtered_text_uri="$2"
          filtered_text_path=$(mktemp)

          # Getting data into the container
          gsutil cp "$text_uri" "$text_path"

          # Running the main code
          grep "$pattern" "$text_path" >"$filtered_text_path"

          # Getting data out of the container
          gsutil cp "$filtered_text_path" "$filtered_text_uri"
        ''',
        placeholders.InputValuePlaceholder('pattern'),
        placeholders.InputUriPlaceholder('text'),
        placeholders.OutputUriPlaceholder('filtered_text'),
    ],
)
```
