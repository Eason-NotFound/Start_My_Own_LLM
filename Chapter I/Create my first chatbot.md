# A. Create my first chatbot

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
2. Connect API
3. Prepare Functions
4. Create Prompt
5. Create Panel
6. For user who can't access to OpenAI

## 1. Prepare installations
We use Openai and Panel, Panel is used to display text and interact with user
** Remember to upgrade openai **

```
!pip install -q openai==1.1.1
!pip install -q panel
!pip install jupyter_bokeh
!pip install --upgrade openai panel
```
## 2. Connect API by creating a client (since we user another server rather than openai)
*** 
```
import openai
import panel as pn
openai.api_key="Your_api_key"
#model = "gpt-3.5-turbo"
model = "gpt-4o-mini"
```

## 3. Prepare functions
### This function is for receiving the message and return output
It simply call openai to have the conversation with you
To call openai, we need these parameters

| Parameters  | Usage  |
|:----------|:----------|
| Model   | The model we want    |
| Message    | Message part of the conversation   |
| Temperature  | Number between 0-2, The smaller the value, the less original the model’s response will be. That determines the imagination of the model    |

The lower the value of temperature, the more similar the result will be for the same inputs

```
#This function will receive the different messages in the conversation,
#and call OpenAI passing the full conversartion.

def continue_conversation(messages, temperature=0):
    response = openai.chat.completions.create(
        model=model,
        messages=messages,
        temperature=temperature,
    )
    return response.choices[0].message.content
```

### This function is for storing the past information
This function collects input and incorporating it into the context or conversation. It is simply like adding text into list.
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
## 4. Create the prompt
It could be seperated into two parts: 
* Indicating how the model should behave(Act like a bot in a icecream shop trying to find out what customers want to buy).
* Give robots the menu.
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
Finally, we use panel to get the user input prompt and put the model to work
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
dashboard # Call the function
```
## 6. Expecailly, for user who can't access to openai
You could look for other server and simply change part 2 and 3

### For part 2, it becames client
```
from openai import OpenAI
import panel as pn
client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://api.gptsapi.net/v1"
)
#model = "gpt-3.5-turbo"
model = "gpt-4o-mini"
```
### For part 3, only change openai to client as well
```
def continue_conversation(messages, temperature=0):
    response = client.chat.completions.create(  # 使用 client 而不是 openai
        model=model,
        messages=messages,
        temperature=temperature,
    )
    return response.choices[0].message.content
```





























