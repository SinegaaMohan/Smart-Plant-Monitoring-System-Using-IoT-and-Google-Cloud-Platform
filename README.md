# AI-Enhanced IOT System for Real Time Diagnosting and Monitoring

## Setting up the Conda Environment

To create a Conda environment identical to the one used in this project, follow these steps:

1. Clone the repository:
   ```bash
   git clone <repository_url>
   cd <repository_directory>
   ```

2. Create the environment using the environment.yml file:
  ```
  conda env create -f environment.yml
  ```
  
3. Activate the Environment:
  ```
  conda activate <environment_name>
  ```
  
  Replace <repository_url> and <environment_name> with your specific values.

  
## Setup for ReactJS

1. Install Nodejs ([Setup instructions](https://nodejs.org/en/download/package-manager/))
2. Install NPM ([Setup instructions](https://www.npmjs.com/get-npm))
3. Install dependencies

```bash
cd frontend
npm install --from-lock-json
npm audit fix
```

## Running the API

### Using FastAPI

1. Get inside `api` folder

```bash
cd api
```

2. Run the FastAPI Server using uvicorn

```bash
uvicorn main:app --reload --host 0.0.0.0
```

3. Your API is now running at `0.0.0.0:8000`

### Using FastAPI & TF Serve

1. Get inside `api` folder

```bash
cd api
```

2. Copy the `models.config.example` as `models.config` and update the paths in file.
3. Run the TF Serve (Update config file path below)

```bash
docker run -t --rm -p 8501:8501 -v path/of/your/project --rest_api_port=8501 --model_config_file=/path/to/configurationfiles
```

4. Run the FastAPI Server using uvicorn
   For this you can directly run it from your main.py or main-tf-serving.py using pycharm run option (as shown in the video tutorial)
   OR you can run it from command prompt as shown below,

```bash
uvicorn main-tf-serving:app --reload --host 0.0.0.0
```

5. Your API is now running at `0.0.0.0:8000`

## Running the Frontend

1. Get inside `api` folder

```bash
cd frontend
```

2. Run the frontend

```bash
npm run dev
```

## Creating the TF Lite Model

1. Run Jupyter Notebook in Browser.

```bash
jupyter notebook
```

2. Open `training/tf-lite-converter.ipynb` in Jupyter Notebook.
3. In cell #2, update the path to dataset.
4. Run all the Cells one by one.
5. Model would be saved in `tf-lite-models` folder.

## Deploying the TF Lite on GCP

1. Create a GCP account
2. Create a Project on GCP (Keep note of the project id).
3. Create a GCP bucket.
4. Upload the model in the bucket.
5. Install Google Cloud SDK.
6. Authenticate with Google Cloud SDK.

```bash
gcloud auth login
```

7. Run the deployment script.

```bash
cd gcp
gcloud functions deploy predict --runtime python310 --trigger-http --memory 1024 --project model-437417
```

8. Your model is now deployed.
9. Use Postman to test the GCF.

# TensorFlow Serving with Docker

This guide explains how to use TensorFlow Serving in a Docker container. Below are the steps to start the container, configure the model, and run TensorFlow Serving.

## Docker Commands

### 1. Start the Docker Container (Interactive Mode)

To start a Docker container in interactive mode, use the following command:

```bash
docker run -it -v /home/punky/ml/tf_gpu/models:/models -p 8605:8605 --entrypoint /bin/bash tensorflow/serving
```

- `-it`: Runs Docker in interactive mode, allowing you to interact with the container through a shell.
- `-v /home/punky/ml/tf_gpu/models:/models`: Mounts the local directory `/home/punky/ml/tf_gpu/models` to the container's `/models` directory.
- `-p 8605:8605`: Maps the host port `8605` to the container's port `8605`.
- `--entrypoint /bin/bash`: Opens a Bash shell inside the container.
- `tensorflow/serving`: The TensorFlow Serving Docker image.

### 2. Start TensorFlow Serving

Once inside the container, you can start TensorFlow Serving by running:

```bash
tensorflow_serving --rest_api_port=<portnumber> --model_config_file=</path/to/config_file> --allow_version_labels_for_unavailable_models
```

- `--rest_api_port=<portnumber>`: Specifies the REST API port where the service will listen (replace `<portnumber>` with the desired port, e.g., `8605`).
- `--model_config_file=</path/to/config_file>`: Specifies the path to your TensorFlow Serving model configuration file.
- `--allow_version_labels_for_unavailable_models`: Allows serving version-labeled models even if some versions are unavailable.

### 3. Start Docker Without Interactive Mode

If you prefer to run the container without interacting with a shell, you can use the following command:

```bash
docker run -t --rm -p 8605:8605 -v /home/punky/ml/tf_gpu/models:/models tensorflow/serving --rest_api_port=8605 --model_config_file=/models/models.config
```

- `-t`: Allocates a pseudo-TTY for better handling of logs.
- `--rm`: Automatically removes the container after it exits.
- `-p 8605:8605`: Maps the host port `8605` to the container port `8605`.
- `-v /home/punky/ml/tf_gpu/models:/models`: Mounts the local directory `/models` to the container's `/models` directory.
- `tensorflow/serving`: The TensorFlow Serving image.
- `--rest_api_port=8605`: Specifies the REST API port where the service will listen.
- `--model_config_file=/models/models.config`: Specifies the path to the model configuration file inside the container.

### 4. Running TensorFlow Serving Without Interactive Mode

Once the container is running in non-interactive mode, TensorFlow Serving will start automatically based on the parameters provided. Make sure your models are configured properly in the `models.config` file.

## Example Model Configuration File

Here's an example of a basic `models.config` file to be used with TensorFlow Serving:

```yaml
model_config_list: {
  config: {
    name: "my_model",
    base_path: "/models/my_model",
    model_platform: "tensorflow"
  }
}
```

- `name`: The name of the model.
- `base_path`: The path where your model is located inside the Docker container (`/models/my_model`).
- `model_platform`: The platform for the model (in this case, TensorFlow).

## Additional Notes

- Ensure your models are stored in `/home/punky/ml/tf_gpu/models` on your local machine, and they are correctly configured in the `models.config` file.
- Adjust the port number (`8605`) and model paths as needed for your environment.
- Use the `--allow_version_labels_for_unavailable_models` flag if you're using model versioning and want to serve models even if some versions are not available.

## Example Workflow

1. **Run Docker in Interactive Mode**:
   ```bash
   docker run -it -v /home/punky/ml/tf_gpu/models:/models -p 8605:8605 --entrypoint /bin/bash tensorflow/serving
   ```
   Once inside the container:
   ```bash
   tensorflow_serving --rest_api_port=8605 --model_config_file=/models/models.config --allow_version_labels_for_unavailable_models
   ```

2. **Run Docker without Interactive Mode**:
   ```bash
   docker run -t --rm -p 8605:8605 -v /home/punky/ml/tf_gpu/models:/models tensorflow/serving --rest_api_port=8605 --model_config_file=/models/models.config
   ```

By following these steps, you can easily serve your TensorFlow models using Docker and TensorFlow Serving.



