# CST8915 - Full Stack Cloud Native Development

## Lab 3: Azure Web App & Azure Static Web App Services

|                  |                              |
| ---------------- | ---------------------------- |
| **Semester**     | Winter 2026                  |
| **Name**         | Mimi Dib                     |
| **Student ID**   | 040829779                    |
| **Date**         | February 12, 2026             |

---

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/ofNjBNLHcv4)

## Repository Links:
- [order-service](https://github.com/mimidib/order-service-lab2-demo/tree/main)
- [product-service](https://github.com/mimidib/product-service-L3P)
- [store-front](https://github.com/mimidib/store-front-lab2)

### Reflection Questions

**1. What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?**

**GitHub Actions couldn't find Node-lts:**
- Node 20-lts wasn't recognized and this runtime wasn't supported for GitHub Actions deployment in Azure. So I asked DeepSeek a few questions regarding this to understand the issue and help find a solution, and the approach I took was the following:
    - Replace the build setup step from using `node-version: '20-lts'` to `node-version: '20.x'`
    - This ensured the build failure could find Node and run as expected

**No explicit startup command:**
- Azure didn't know how to start order-service, and since we don't include an `npm start` command in the package.json, this meant we couldn't launch the application.
- I removed the `npm install`, `npm run build`, and `npm run test` lines from the workflow YAML file Azure created. DeepSeek told me Azure was able to recognize there were no `node_modules` folder uploaded and no build output, so it default enabled `SCM_DO_BUILD_DURING_DEPLOYMENT=true` for me. However, it still had no way to run the application without an `npm start` or `node index.js` command explicitly. DeepSeek explained that Azure then sees package.json without a `start` script and checks the main field `"main": "index.js"` and automatically runs `node index.js` to start the app. In editing the workflow file, these are specifically the lines I removed:
```yaml
- name: npm install, build, and test
    run: |
        npm install
        npm run build --if-present
        npm run test --if-present
```
- After this, I noticed the app still wasn't actually running by looking at the log stream:
[Azure order-service Log Stream](npm-run.png)
So I went and added it to my package.json file. I added to scripts: "start": "node index.js" because it must automatically not actually look at the main field as DeepSeek said.

- Then I noticed the build passed but the application couldn't start after looking at the logs. I had actually needed to explicitly add a build step in the package.json that was a dummy step to do nothing, and then to start using the start script with: `"build": "echo \"No build step required\" && exit 0"`. 

**Store-front doesn't deploy due to policy issue**
As spoken about over Teams, Ramy and I established a policy issue hindered the deployment of store-front on Azure Static Web Apps. This meant I had to deploy it from the RabbitMQ VM. But to establish my understanding of the workflow file change using the `VUE_APP` env variables, this is what my `build_and_deploy_job` section would look like:  

```yaml
jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    env: # Declare environment variables here
      VUE_APP_ORDER_SERVICE_URL: https://cst8915-order-service-node20-axcdbeaqe0fbcjhw.canadacentral-01.azurewebsites.net/
      VUE_APP_PRODUCT_SERVICE_URL: https://product-service-python-dqhbdpfvbtggarh0.canadacentral-01.azurewebsites.net/
```

**2. How does deploying microservices on Azure Web App Service differ from running them locally?**  

What differs from deploying an application on Azure Web App Service from running locally is all we need to do is tell Azure what repository to run the code from, and it will handle the deployment and even create the workflow files itself and run them on demand. We don't need to worry about the runtime, ubuntu server (OS) or the actual VM creation or maintaining the OS. We also don't need to worry about scaling, as it is completely handled in the backend by the cloud provider. All we do is tell Azure what repo we want to deploy, add environemtn variables, ensure the ports are accessible and its running!

3. Why is it important to use environment variables for configurations in a cloud environment?  

Utilizing environment variables (Config factor) allows us to keep the codebase unchanged and independent across environments, and adds security by not having it hardcoded in the codebase. It allows us to deploy and scale applications rapidly while being able to keep a fast deployment cycle for new feature development. Also, for any future changes of these values like connection strings or the like, there are no changes to the application code itself needed; instead, just changing these values in the `.env` file, or in Web Apps, on the environment variables gui or for static web apps changing it in the workflow YAML will suffice without issues.


