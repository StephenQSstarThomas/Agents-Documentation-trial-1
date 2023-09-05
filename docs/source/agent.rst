AI Autonomous Agents
=====================

Brief Introduction
------------------

AI Autonomous Agents are intelligent entities capable of perceiving their environment, making decisions, and taking actions without direct human intervention. As human society rapidly develops, AI Autonomous Agents find applications in diverse circumstances.

Demonstrations & Attributes
---------------------------

Basic Codes
~~~~~~~~~~

.. code:: python

    class Agent:
        """
        Auto agent, input the JSON of SOP.
        """
        
        # Agent should have args: agents,states
        def __init__(self, name, agent_state_roles, **kwargs) -> None:
            self.state_roles = agent_state_roles
            self.name = name
            self.LLMs = kwargs["LLMs"]
            self.is_user = kwargs["is_user"]
            
            self.long_term_memory = []
            self.short_term_memory = ""
            self.current_state = None
            self.agent_dict = {
                "name": name,
                "style" : kwargs["style"],
                "current_roles": "",
                "long_term_memory": self.long_term_memory,
                "short_term_memory": self.short_term_memory,
                
            }

    # Remark:
    # content : The Agent's whole prompt. See clear definition and explanation in our Components Part!
    # agent_state : Chatting memories of the Agent. We will also manifest the methods which target the chatting history in our SOP System Part. All of the attached attributes, such as roles, names, etc will be thoroughly explained in the SOP part.

Methods
~~~~~~~

From_config
~~~~~~~~~~~

.. code:: python

    def from_config(cls, config_path):
        with open(config_path) as f:
            config = json.load(f)
        roles_to_names = {}
        names_to_roles = {}
        agents = {}
        user_names = json.loads(os.environ["User_Roles"]) if "User_Roles" in os.environ else []
        for agent_name, agent_dict in config["agents"].items():
            agent_state_roles = {}
            agent_LLMs = {}
            for state_name, agent_role in agent_dict["roles"].items():
                if state_name not in roles_to_names:
                    roles_to_names[state_name] = {}
                if state_name not in names_to_roles:
                    names_to_roles[state_name] = {}
                roles_to_names[state_name][agent_role] = agent_name
                names_to_roles[state_name][agent_name] = agent_role
                agent_state_roles[state_name] = agent_role
                current_state = config["states"][state_name]
                LLM_type = (
                    current_state["agent_states"][agent_role]["LLM_type"]
                    if "LLM_type" in current_state["agent_states"][agent_role]
                    else "OpenAI"
                )
                if LLM_type == "OpenAI":
                    if "LLM" in current_state["agent_states"][agent_role]:
                        agent_LLMs[state_name] = OpenAILLM(
                            **current_state["agent_states"][agent_role]["LLM"]
                        )
                    else:
                        agent_LLMs[state_name] = OpenAILLM(model="gpt-3.5-turbo-16k-0613", temperature=0.3, log_path=f"logs/{agent_name}")
            agents[agent_name] = cls(
                agent_name,
                agent_state_roles,
                LLMs=agent_LLMs,
                is_user=agent_name in user_names,
                style=agent_dict["style"]
            )
        return agents, roles_to_names, names_to_roles

    # Remark:
    # The from_config method starts the agent according to the given attributes and data.

Act
~~~

.. code:: python

    def act(self):
        """
        return actions by the current state
        """
        current_state = self.current_state
        system_prompt, last_prompt, res_dict = self.compile()
        chat_history = self.agent_dict["long_term_memory"]

        current_LLM = self.LLMs[current_state.name]

        response = current_LLM.get_response(
            chat_history, system_prompt, last_prompt, stream=True
        )
        return {
            "response": response,
            "res_dict": res_dict,
            "role": self.state_roles[current_state.name],
            "name": self.name,
        }

    # Remark:
    # The act method generates and outputs the response of the Agent. Detailed explanations on particular attributes will be shown afterwards.

Step
~~~~

.. code:: python

    def step(self, current_state, environment):
        """
        return actions by current state and environment
        """
        current_state.chat_nums += 1
        self.current_state = current_state

        # First update the information according to the current environment
        self.observe(environment)
        if self.is_user:
            response = input(f"{self.name}:")
            response = f"{self.name}:{response}"
            return {
                "response": response,
                "is_user": True,
                "role": self.state_roles[current_state.name],
                "name": self.name,
            }
        else:
            current_history = self.observe(environment)
            self.agent_dict["long_term_memory"].append(current_history)
            return self.act()

    # Remark:
    # Closely related to the act method, the step method updates the current circumstance and then returns the response of an Agent. Detailed explanations on particular attributes will be shown afterwards.

Compile
~~~~~~~

.. code:: python

    def compile(self):
        """
        get prompt from state depend on your role
        """
        current_state = self.current_state
        self.agent_dict["current_roles"] = self.state_roles[current_state.name]
        current_state_name = current_state.name
        self.agent_dict["LLM"] = self.LLMs[current_state_name]
        components = current_state.components[self.state_roles[current_state_name]]

        system_prompt = self.current_state.environment_prompt
        last_prompt = ""

        res_dict = {}
        for component in components.values():
            if isinstance(component, (OutputComponent, LastComponent)):
                last_prompt = last_prompt + "\n" + component.get_prompt(self.agent_dict)
            elif isinstance(component, PromptComponent):
                system_prompt = (
                    system_prompt + "\n" + component.get_prompt(self.agent_dict)
                )
            elif isinstance(component, ToolComponent):
                response = component.func(self.agent_dict)
                if "prompt" in response and response["prompt"]:
                    last_prompt = last_prompt + "\n" + response["prompt"]
                self.agent_dict.update(response)
                res_dict.update(response)
        return system_prompt, last_prompt, res_dict

    # Remark:
    # The Compile method reaches for the current role, and returns the action of a certain agent state.

Observe
~~~~~~~

.. code:: python

    def observe(self, environment):
        """
        get new memory from environment
        """
        MAX_CHAT_HISTORY = eval(os.environ["MAX_CHAT_HISTORY"])
        current_state = self.current_state
        current_role = self.state_roles[current_state.name]
        current_component_dict = current_state.components[current_role]

        if environment.environment_type == "compete":
            current_long_term_memory = environment.shared_memory["long_term_memory"][environment.current_chat_history_idx:]
            current_chat_embbedings = environment.shared_memory["chat_embeddings"][environment.current_chat_history_idx:]
        else:
            current_long_term_memory = environment.shared_memory["long_term_memory"]
            current_chat_embbedings = environment.shared_memory["chat_embeddings"]

        # total memory observed
        current_memory = "Here's what you need to know(Remember, this is just information, Try not to repeat what's inside):\n<information>\n"

        # relevant_memory
        relevant_memory = "The relevant chat history are as follows:\n<relevant_history>"
        query = current_long_term_memory[-1]

        key_history = get_key_history(
            query,
            current_long_term_memory[:-1],
            current_chat_embbedings[:-1],
        )
        for history in key_history:
            relevant_memory += (
                f"{history.send_name}({history.send_role}):{history.content}\n"
            )

        relevant_memory += "<relevant_history>\n"
        self.agent_dict["relevant_history"] = relevant_memory

        # get new conversation
        last_conversation_idx = -1
        for i, history in enumerate(current_long_term_memory):
            if history.send_name == self.name:
                last_conversation_idx = i

        if last_conversation_idx == -1:
            new_conversation = current_long_term_memory
        elif (
            last_conversation_idx
            == len(current_long_term_memory) - 1
        ):
            new_conversation = []
        else:
            new_conversation = current_long_term_memory[
                last_conversation_idx + 1 :
            ]

        # get chat history from new conversation
        conversations = Memory.get_chat_history(new_conversation)

        if len(current_long_term_memory) % MAX_CHAT_HISTORY == 0:
            # get summary
            summary_prompt = (
                current_state.summary_prompt[current_role]
                if current_state.summary_prompt
                else f"""your name is {self.name}, your role is {current_component_dict["style"].role}, your task is {current_component_dict["task"].task}.\n"""
            )
            summary_prompt += """Please summarize and extract the information you need based on past key information \n<information>\n {self.short_term_memory} and new chat_history as follows: <new chat>\n"""
            summary_prompt += conversations + "</new chat>\n"
            response = self.LLMs[current_state.name].get_response(None, summary_prompt)
            summary = ""
            for res in response:
                summary += res
            self.agent_dict["short_term_memory"] = summary
            self.short_term_memory = summary

            # memory = relevant_memory + summary + history + query
        current_memory += (relevant_memory + \
            f"The previous summary of chat history is as follows :<summary>\n{self.short_term_memory}\n</summary>.\
            The new chat history is as follows:\n<new chat> {conversations}\n</new chat>\n\
            </information>,\
            You especially need to pay attention to the last query<query>\n{query.send_name}({query.send_role}):{query.content}\n</query>\n")

        return {"role": "user", "content": current_memory}

    # Remark:
    # The Observe method is the core method of an agent. It updates and reads the current environment, including the chatting history and the basic information, and returns particular actions for the agent.

Examples
~~~~~~~~

We provide various types of Agents in our QuickStart part. You can also train your OWN Agent in a customized style!
