Thank you for your interest in testing the Evidently Monitoring interface! 

We will release this feature to general availability by the end of July. We’ll be glad to see you at the public virtual [Launch event on July 27](https://www.evidentlyai.com/launch-day). 

The goal of this alpha testing is to collect feedback and help us shape the roadmap.  

This document includes the following:

| Item | Comment |
|----------------------------------------------------------------------------------------|-------------------------------------------------|
| 1. Instructions to launch the pre-built example with Evidently Monitoring UI locally.  | Estimated time: 10 min.                         
| 2. [OPTIONAL] Instructions on how to modify the example.                               | Only if you want to poke around.                |
| 3. Explanation of how the monitoring workflow will work in practice.                   | Estimated reading time: 5 min.                  |
| 4. Questions                                                                           | List of questions we’d appreciate feedback on.  |

# 1. Launch the UI example 
It includes a demo project with pre-built dashboards and 20 daily snapshots: as if you ran daily batch inference, and logged the results every day.  

![](https://github.com/elenasamuylova/evidently-misc/blob/main/img/ui_basic-min.png)

To run the example of Evidently UI service follow the steps below.

**Step 1**: (optional, but highly recommended) Create virtual environment and activate it.
```python
pip install virtualenv
virtualenv venv
source venv/bin/activate
```

**Step 2**: Clone evidently repo
```python
git clone git@github.com:evidentlyai/evidently.git
```

**Step 3**: Check out branch service_ui
```python
cd evidently
git checkout feature/evidently_service_ui
```

**Step 4**: Build the project
```python
pip install -e .
```

**Step 5**: Run the command to generate an example workspace with a project and some reports and test suites.
```python
evidently generate_workspace --workspace ./workspace
```

**Step 6**: Run evidently UI service
```python
evidently ui --workspace ./workspace --port 8080
```

**Step 7**: Access Evidently UI service in your browser

Go to the `localhost:8080`

>What you can do in the interface:
>
>**Dashboards** show metrics in time. Evidently automatically parses them from reports and test suites. Zoom in and out and explore.
>
>**Reports** and **Test suites** show individual Evidently reports and test suites, as if logged daily. You can explore and download.

# 2. Change the example (Optional)

To update the example, e.g. edit dashboard, reports or test suites follow steps below. 
**Note**: there is no detailed documentation yet, but it’s possible to follow along by example. 

**Step 1**: Open `generate_workspace.py` script and edit it. Save your changes.

The path is `src/evidently/ui/generate_workspace.py`

What happens in this script:
* We import the usual Evidently Reports and Test Suites. They define which metrics will be available for monitoring. -> You can **specify monitoring contents** using any existing Evidently presets, metrics, or tests.
* We **create a workspace** -> You can give it a custom name. 
* We **create a project**. Each project corresponds to a different model you monitor. -> You can give a custom name to your project.
* We import a demo dataset (“adult”) and define a script to generate 20 daily **snapshots** to imitate production batch inference -> You can **replace it for your data** and generate a few snapshots. Important: you must store the snapshots inside a specific project.
* We **add panels to the project**. A panel corresponds to a single visual dashboard that appears in the “dashboards” section. -> You can define your composition:
   * You can create as many dashboards as you like and give custom names.
   * You can only create a dashboard for the metrics that were logged. 
   * You can show multiple metrics on the same dashboard. 
   * You can choose between the “counter” and “plot” types.

What is a snapshot:
A **snapshot** is a single “log” instance, e.g., a daily log. You can convert a snapshot to the visual Evidently Report or its structured JSON / Python dictionary version.

**Step 2**: Delete previously generated workspace.
```python
cd src/evidently/ui/
rm -r workspace
```

**Step 3**: Run the script `generate_workspace.py` you created at step 1 to create a new workspace, generate snapshots and dashboards.

**Step 4**: Run evidently ui service (change the path if you renamed the workspace)
```python
evidently ui --workspace ./workspace --port 8080
```

**Step 5**: Access Evidently UI service in your browser. Go to the `localhost:8080`

# 3. How will it work in production?

If you have batch model inference, e.g., daily, you can log the Evidently `snapshots` after any meaningful step in the pipeline. Just like you run the usual Evidently Reports or Test Suites.

![](https://github.com/elenasamuylova/evidently-misc/blob/main/img/evidently_batch-min.png)

You can store the JSON snapshots in an object storage or local folder. 

**Note**: a snapshot is different from using the usual `json()` or `save_json(“file.json”)`. When using the usual JSON export, you generate a structured output with limited information. You cannot convert JSON back to HTML. With `snapshot`, you generate a comprehensive summary and can restore the output in any available Evidently format without accessing the initial raw data. 

The snapshot functionality is already available. You can save the snapshot after you create any Evidently Report or Test suite, use the `save` function and specify the path.
```python
data_drift_dataset_report._save(os.path.join(PATH, f"data_drift_profile.json"))
```

To load the snapshot back, use the “load” function. 
```python
restored_report = Report._load(os.path.join(PATH, f"data_drift_profile.json"))
Restored_report
```

There is also an API with a workspace add method used in the script above. It directly adds a snapshot to the workspace.

**Note**: the below functionality is not yet released and will not be part of the initial release.  

If you deploy your ML model as a service, you will be able to POST model logs to a collector service that will abstract away the computation of snapshots. You will be able to define the batch size or metric computation frequency using a config file.

![](https://github.com/elenasamuylova/evidently-misc/blob/main/img/evidently_rt-min.png)

# What’s next
We are currently working on the following:
* Documentation
* Add a selection of time ranges in the UI (to allow only showing, e.g., “last 7 days”).
* Give the ability to add and delete dashboard panels in the interface.
* Add pre-built tabs to the interface. Right now, you define and create each dashboard on your own. We want to automatically show tabs with predefined visualizations to summarize “Data drift,” “Data Quality,” and “Model Quality.” 

# 4. Questions: help us make Evidently better! 

* **Clarity**: is anything unclear in this workflow? What should we explain better when documenting and showing to users?
* **Blockers and lacking features**: can you use this monitoring UI in your workflow? If not, which functionality we must add for you to be able to use it?
* **Adding dashboards from the user interface VS defining in the code**. Do you want to add dashboards directly from UI, or defining them via code configuration is OK? How important is the definition of dashboards in UI to start using monitoring?
* **Logging a limited set of metrics VS “logging it all”**. There are two possible approaches: 1. Select and log a predefined list of metrics. 2. Log “as much as can be useful” upfront. The second approach simplifies debugging and adding extra dashboards. How do you expect to use Evidently UI - will you manually pick metrics, or do you want to use logging presets?
* **Adding tabs**. We are considering adding the following pre-built tabs:
  * Data quality
  * Data drift
  * Model quality 
Which tabs would you like to see and use?

* **How important is integration with ML service**, or can you start using Evidently UI with a batch workflow? Which orchestrator (Kubeflow, Airflow, Prefect, Databricks pipelines, etc.) or pipeline framework (Metaflow) do you plan to use it with?

* Anything else you want to share!






