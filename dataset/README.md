# HuggingFace Dataset Scripts
This folder contains four scripts to generate the `jsonl` datasets that are currently released on [HuggingFace](https://huggingface.co/datasets/NetConfEval/NetConfEval).

By default, the scripts will generate the same dataset as the ones used by the evaluation scripts in the `netconfeval` folder.

You can customize the generation by modifying the scripts.

## Usage
To use the scripts, the Python packages: `sortedcontainers`, `pytest`, `kathara`, and `docker` are required.

### Translating High-Level Requirements to a Formal Specification Format
This script will generate the dataset that can be used to evaluate `step_1_formal_spec_translation.py` and `step_1_function_call.py`.

Run the following command:
```bash
python3 generate_step_1_dataset.py 
```

The result will be saved in `datasets/step_1_spec_translation.jsonl` by default. You can change the folder by specifying the `--results_path` argument.

You can change the input Config2Spec policy file (in CSV) by specifying the `--policy_file` argument.

By default, the script will generate a dataset with five iterations, if you want to change it, specify the `--n_runs` argument.

You can find the policies and the associated batch sizes in the file, under the `policies_to_batch_sizes` Dict. You can change it as required.

### Conflict Detection
This script will generate the dataset that can be used to evaluate `step_1_formal_spec_conflict_detection.py`.
It will insert a "simple conflict" in each even batch (0, 2, ...).

Run the following command:
```bash
python3 generate_step_1_conflict_dataset.py 
```

The result will be saved in `datasets/step_1_spec_conflict.jsonl` by default. You can change the folder by specifying the `--results_path` argument.

You can change the input Config2Spec policy file (in CSV) by specifying the `--policy_file` argument.

By default, the script will generate a dataset with five iterations, if you want to change it, specify the `--n_runs` argument.

You can find the policies and the associated batch sizes in the file, under the `policies_to_batch_sizes` Dict. You can change it as required.


### Developing Routing Algorithms
This script will generate the dataset that can be used to evaluate `step_2_code_gen.py`.

Run the following command:
```bash
python3 generate_step_2_dataset.py
```

The result will be saved in `datasets/step_2_code_gen.jsonl` by default. You can change the folder by specifying the `--results_path` argument.

The dataset contains both the input user prompt (without preliminary system prompts) in the `input` column and a series of test cases to run on the generated code in the `tests` column.

To run the tests, you need to JSON decode the `tests` folder. This will give you a dict with an incremental index as key and the test body as value.
It is recommended to run the tests in order, following the index key. You need the `pytest` package to run the tests. 

After extracting the test body:
- Replace the `# ~function_code~` placeholder with the code generated by the LLM;
- Save the resulting string into a `.py` file in your filesystem, for example `test_file.py`;
- Run `python3 -m pytest --lf --tb=short test_file.py -vv`.

The above procedure is implemented in NetConfEval through the `netconfeval/verifiers/step_2_verifier_detailed.py` class.

### Generating Low-level Configurations
This script will generate the dataset that can be used to evaluate `step_3_low_level.py`.
You need the `kathara` and `docker` packages to generate the dataset.

Run the following command:
```bash
python3 generate_step_3_dataset.py
```

The result will be saved in `datasets/step_3_low_level.jsonl` by default. You can change the folder by specifying the `--results_path` argument.

The generation will run a Docker container (through Kathará) with FRRouting, and will convert the configurations in a deterministic output using the `vtysh` command line.

The dataset contains both the input user prompt (without preliminary system prompts) in the `input` column and the corresponding configuration for each device in the `result` column.

To compare the generated LLM configuration with the expected one, we suggest to:
- JSON decode the `result` column, this will give you a Dict with the device name as key and the expected configuration as value (in string);
- Take the LLM output and, for each device, run the same formatting command in the `vtysh` using the FRRouting container;
- Compare the two outputs using `difflib.SequenceMatcher`.

The above procedure is implemented in NetConfEval in the `netconfeval/step_3_low_level.py` script.