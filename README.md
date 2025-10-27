### Exercise 7: Enterprise Agentic AI Orchestration at Scale

This lab focuses on integrating WxO Agents with Langflow, and enables participants to explore Docling, an open-source document extraction tool powered by Vision-Language Model (VLM) technology.

*Please make sure to install any required programs and register to required platform on your laptop before joining the event. Note that you will receive the link to the GitHub repository on the day of the event.

**IMPORTANT: If you are Windows user. Any command please run as admin**

### 1. Setup IBM Cloud free trial version
1. Please access the following link:
https://www.ibm.com/products/watsonx-orchestrate and click on `Try it for free`
![alt text](assets/images/image.png)

2. please fill in your email address can be personal email address and other information to register for watsonx Orchestrate
![alt text](assets/images/image-1.png)


3. After registering you should get a code to your email.
![alt text](assets/images/image-2.png)

4. Please add the code to register for free trial
![alt text](assets/images/image-3.png)

5. Once registered please deploy the trial in `us-east` region
![alt text](assets/images/image-4.png)

![alt text](assets/images/image-5.png)

6. You will be directed to: https://dl.watson-orchestrate.ibm.com/. Afterwards please, login with the IBMid and password you signed up with.
![alt text](assets/images/image-7.png)

7. Your instance have been successfully provisioned once you landed here.
![alt text](assets/images/image-8.png)

### 2. Add Gemini
4. Our goal is to add `google/gemini-2.5-flash` to watsonx Orchestrate. Please visit https://aistudio.google.com and click generate an API key to get your API key.

![alt text](assets/images/getapikey.png)
![alt text](assets/images/success.png)

NOTE:
If you are unable to create api key, it means you haven't enabled Gemini API in Google cloud console and created a project. Hence please, enable gemini API and create a project.

![alt text](assets/images/unabletocreatekey.png)
Enabling Gemini API
![alt text](assets/images/enablegemini.png)
Creating Project
![alt text](assets/images/create-gcpproject.png)

### 3. Setup forward port
1. Go to https://ngrok.com/download/windows?tab=download register and install. Only if you does not have Ngrok installed yet.
(Required only non-installed Ngrok)
2. Run `ngrok http 7860`

### 4. Setup Critic Agent on Langflow

![alt text](assets/step5/image.png)

1. Go to Langflow. If you never start it, do with command `docker run -p 7860:7860 langflowai/langflow:latest`

![alt text](assets/step5/image2.png)

2. Import project with the given JSON file (Workshop Flow.json)
3. In the flow chage the model to the provider you have API key, and input your API key (can replicate from previous workshop where you can use ChatGPT).
4. Forward port of this application (should be 7860)

### 5. Setup Master Agent and Docling Agent on watsonx Orchestrate

0. run `cd orchestrate` then `uv sync`. In case you don't have uv do `pip install uv`
- For windows, run `.\.venv\Scripts\activate.bat`. For macOS, `source .venv/bin/activate`.

![alt text](assets/step6/image.png)

1.Go to watsonx Orchestrate connections

![alt text](assets/step6/image_1.png)
![alt text](assets/step6/image_2.png)

2. Click on Hamburger bar on top left. Click "Manage" and "Connections". Add the Connection with same setting as images above (ID and name should be `gg_creds`). Click on "Connect" and then click "Next". Click "Paste draft configuration" then "Add connection".

3. Setup WxO instance
3.1 Go to profile on the top-right. Click "Settings" and select tab "API details". Save the "Service instance URL" with you. Also, Click "Generate your API key" and save it on your note.
3.2 Back to VSCode where you ran cd to orchestrate. Run `orchestrate env add -n workshop-ai -u <Your Service instance URL>` and run `orchestrate env activate workshop-ai` then paste your saved API key.

5. On your terminal run `orchestrate models add --name "google/gemini-2.5-flash" --app-id gg_creds`

6. Import agent with command `orchestrate agents import -f agents/docling_agent.yml` and `orchestrate agents import -f agents/master_agent.yml`

![alt text](assets/step6/image6.png)

6. Go to "docling_agent" on watsonx Orchestrate. Click "Add tool". Click "Add from file or MCP server". Click "Import from MCP server". Click "Add MCP server".

7. Put service name as "docling_mcp". Put the command `uvx mcp-proxy https://docling-server-large.21oo25foed0i.us-east.codeengine.appdomain.cloud/sse`. Leave other fields as it is. Click "Connect" and "Done". Toggle open for all tools you see on the screen, all pages.

![alt text](assets/step6/image8.png)
![alt text](assets/step6/image9.png)

8. Go to Langflow project you imported. Click "Share" and select "MCP Server". Copy the url as shown on the 2nd image and save it on your note. 

9. Go to Toolset of master_agent. Now, we will add critic_agent of Langflow via MCP. Again, you will click on "Add tool", "Add from file or MCP server", and "Import from MCP server". Click "Add MCP server". Put "Server name" as `langflow`. Replace URL you copy from step 10 from (http://localhost:7860) with (Your forwarded address of port 7860). Put "Install command" as `uvx mcp-proxy <Your fixed url>`. Click "Connect" and "Done". Toggle open for all tools you see on the screen, all pages. 

### 6. Test master_agent
```
Q1: Help me create docling document of this url https://www.infineuminsight.com/media/2601/emea-fundamentals-of-engine-design-and-operation.pdf
A2: <No reference>
```
```
Q2: Two categories of getting air into engine?
A2: Slide 18
```
```
Q3: I think it should be Artificially aspirated rather than Naturally aspirated. Please update the document.
A3: <If somebody changed it, you may ask the agent to change any content you like>
```
```
Q4: How should I improve this document?
A4: <No reference should forward to Langflow>
```

