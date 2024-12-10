These steps explain how to connect a Python virtual environment with Jupyter. This is primarily useful when you're using a hosted Jupyter instance, for example on Intel Tiber Cloud. 

1. Open a terminal and create a new virtual environment: 
  `python3 -m venv my_env` and activate it: `source my_env/bin/activate`
2. Upgrade pip and install ipykernel: `pip install --upgrade pip` and `pip install ipykernel`
3. (This is the key step) Create a Jupyter kernel connected to the virtual environment:
`python -m ipykernel install --user --name my_kernel`
4. Now `my\_kernel` should be visible in the notebooks launcher (see this image, where the kernel name is `vllm`). You may need to refresh the browser window.

![image](https://github.com/user-attachments/assets/9c205f04-135c-4450-977a-92c8783dbd39)

5. You can now create a new notebook from the launcher. To change an existing notebook to use the `my\_kernel` kernel, you can choose Kernel-\>Change Kernel in Jupyter, choose the kernel you created in step 3, and save the notebook
