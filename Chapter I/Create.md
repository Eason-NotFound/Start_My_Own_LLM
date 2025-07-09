# Create my first chatbot

| Roles  | Meaning  | 
|:----------|:----------|
| System   | Give model the basic information we want   | 
| User   | Phases from user   | 
| Asistent   | Responses returned by model   | 

Things we need to achieve:
1. Get answers from OpenAI
2. Store the memory

First, How to get answers from OpenAI --> Call openai.ChatCompletion.create

Second, How to store the memory --> Append conversation into context (The model has no inner storage place)

## Menu
1. Installation
2. Connect Api
3. Prepare Functions
4. Create System
5. Create Panel

## 1. Prepare installations
We use Openai and Panel, here we need openai to be the new version since we are using other server to touch openai (For users who can't access to openai directly)

```
!pip install -q openai==1.1.1
!pip install -q panel
!pip install jupyter_bokeh
!pip install --upgrade openai panel
```
## 2. Connect API by creating a client (since we user another server rather than openai)
```
from openai import OpenAI
import panel as pn
client = OpenAI(
    api_key="sk-G2de37624fc053aca5294b5360ed24c64f8f16db4e1nYepO",
    base_url="https://api.gptsapi.net/v1"
)
#model = "gpt-3.5-turbo"
model = "gpt-4o-mini"
```

## 3. Prepare functions
### This function is for receiving the message and return output
```
#This function will receive the different messages in the conversation,
#and call OpenAI passing the full conversartion.
    
def continue_conversation(messages, temperature=0):
    response = client.chat.completions.create(  # 使用 client 而不是 openai
        model=model,
        messages=messages,
        temperature=temperature,
    )
    return response.choices[0].message.content
```

### This function is for storing the past information
```
def add_prompts_conversation(_):
    #Get the value introduced by the user
    prompt = client_prompt.value_input
    client_prompt.value = ''

    #Append to the context the User promnopt.
    context.append({'role':'user', 'content':f"{prompt}"})

    #Get the response.
    response = continue_conversation(context)

    #Add the response to the context.
    context.append({'role':'assistant', 'content':f"{response}"})

    #Update the panels to show the conversation.
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600)))

    return pn.Column(*panels)
```
## 4. Create the system
```
#Creating the system part of the prompt
#Read and understand it.

context = [ {'role':'system', 'content':"""
You work collecting orders in a delivery IceCream shop called
I'm freezed.

First welcome the customer, in a very friedly way, then collects the order.

Your instuctions are:
-Collect the entire order, only from options in our menu, toppings included.
-Summarize it
-check for a final time if everithing is ok or the customer wants to add anything else.
-collect the payment, be sure to include topings and the size of the ice cream.
-Make sure to clarify all options, extras and sizes to uniquely
identify the item from the menu.
-Your answer should be short in a very friendly style.

Our Menu:
The IceCream menu includes only the flavors:
-Vainilla.
-Chocolate.
-Lemon.
-Strawberry.
-Coffe.

The IceCreams are available in two sizes:
-Big: 3$
-Medium: 2$

Toppings:
-Caramel sausage
-White chocolate
-melted peanut butter
Each topping cost 0.5$

"""} ]
```
## 5. Create the panel
```
pn.extension()

panels = []

client_prompt = pn.widgets.TextInput(value="Hi", placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="talk")

interactive_conversation = pn.bind(add_prompts_conversation, button_conversation)

dashboard = pn.Column(
    client_prompt,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True),
)

#To talk with the chat push the botton: 'talk' after your sentence.
dashboard
```
































