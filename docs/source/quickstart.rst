😄Getting Started with Fun
=========================

Try our demo in your terminal 

1. Open your terminal 🖥️

2. Get the Repository 📦
   ::
   
      git clone https://github.com/aiwaves-cn/agents.git

3. Install the requirements 🛠️
   ::
   
   pip install -r requirements.txt

4. Set the config 🛠️
.. code-block:: json

   Modify example/{Muti|Single_Agent}/{target_agent}/config.json
   // only used for shopping assistant
   {
       MIN_CATEGORY_SIM  =  "0.7"  ##Threshold for category matching
       TOY_INFO_PATH  = "[\"your_path1\",\"your_path2_\"......]" #Path to the product database
       FETSIZE  =  "5" #Number of recommended products at a time

       #for all agents
       API_KEY  =  #Your API KEY
       PROXY  =  #Your proxy
       MAX_CHAT_HISTORY  =  "8" #Longest History
       User_Names = "[\"{user_name}\"]" # the name of agents which you want to play  
   }

   Notice that if you want to use WebSearchComponent, you also need set the config!
   "WebSearchComponent": {
       "engine_name": "bing",
       "api": {
           "google": {
               "cse_id": "Your cse_id",
               "api_key": "Your api_key"
           },
           "bing": "Your bing key"
       }
   }

Deploy our demo on the backend:point_down:

1. Prepare your front-end webpage🌐
2. Deploy🚀
   Please refer to run.py for details
   We used fast_api to deploy🌶️

.. code-block:: bash

   cd examples
   python run_backend.py --agent Single_Agent/shopping_assistant/muti_shop.json --config Single_Agent/shopping_assistant/config.yaml  --port your_port --router your_api_router

Get started with our Agents!

🧠 How to write a modulized JSON file?

Preview
~~~~~
In this passage, we will show you how to write a modulized JSON file, which is of vital significance in generating the Agents.

Part 0: Template
~~~~~~~~~~~~~~~~
The following codes are a typical template for wrting JSON Files.(Please refer to template.py)

.. code-block:: json

   ## default { "temperature": 0.3, "model": "gpt-3.5-turbo-16k-0613","log_path": "logs/{your name}"}
   LLM = {
       "temperature": 0.0,
       "model": "gpt-3.5-turbo-16k-0613",
       "log_path": "logs/god"
   }

   Agents = {
       "Lilong" : {
           "style" : "professional",
           "roles" : {
               "company" : "coder",
               "state2" : "role2",
           },
       "name2" : {   
           "style" : "professional",
               "roles" : {
                   "company" : "coder",
                   "state2" : "role2",
               },
           }
       }
   }

   # indispensable parameter:  "controller_type"（"order","random","rule"）
   controller = {
       "controller_type": "order",
       "max_chat_nums" : 12,
       "judge_system_prompt": "",
       "judge_last_prompt": "",
       "judge_extract_words": "end",
       "call_system_prompt" : "",
       "call_last_prompt": "",
       "call_extract_words": ""
   }

   Agent_state = {
       "role": {
       "LLM_type": "OpenAI",
       "LLM": LLM,
       "style": {
           "role": "Opening Advocate for the Affirmative",
           "style": "professional"
       },
       "task": {
           "task": ""
       },
       "rule": {
           "rule": ""
       }
   },
   }

   # indispensable parameter:  "name" and "agent_states"
   State = {
       "controller": controller,
       "begin_role": "",
       "begin_query": "",
       "environment_prompt": "",
       "name": "state1",
       "roles": ["role1","role2"],
       "LLM_type": "OpenAI",
       "LLM": LLM,
       "agent_state" : Agent_state,
   }

   States = {
       "end_state":{
               "name":"end_state",
               "agent_states":{}
           },
       "state1" : State
   }

   # default finish_state_name is "end_state"
   SOP = {
       "config" : {
       "API_KEY" : "Your key",
       "PROXY" : "Your PROXY",
       "MAX_CHAT_HISTORY" : "5",
       "User_Names" : "[\"alexander\"]"
       },
       "environment_type" : "competive",
       "LLM_type": "OpenAI",
       "LLM" :LLM,
       "root": "state1",
       "finish_state_name" : "end_state",
       "relations": {
           "state1": {
               "0": "state1",
               "1": "state2"
           },
           "state2":{
               "0":"state2",
               "1":"end_state"
           }
       },
       "agents": Agents,
       "states": States,
   }
~~~~~


   (written by JSON master longli)

Part 1: Remark on some of the attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


- SOP consists of State and Agent.

- State: The basic unit of SOP can be considered as a task or a scene, that is, a place where all Agents need to work together. It stores the tasks that agents with different identities need to complete.
  (Note: The task is given to the agent with a specific identity rather than the agent with a specific name)

  States: Stores all states.
  - name: The ONLY signal of one particular State in a certain SOP.
  - Controller: Determine whether the current state ends based on its system prompt and last prompt, which State should be entered next, and which Agent the task should be assigned to.
  - begin_role & begin_query: The Agent who should speak and the corresponding words when entering the scene for the first time.
  - environment_prompt: Responsible for explaining the overall situation of the current State; it will be added before the system prompt of all Agents in the current scene.
  - roles: All roles in the current state.

- Agent_state: Component of different agents in this state.
  - style & task & rules & demonstration & CoT & Output: Please refer to PromptComponent part, which is aforementioned.
  - KnowledgeBaseComponent: Please refer to ToolComponent part, which is also mentioned above.

- SOP: Fundamental attributes of the SOP graph.
  - active_mode: Decide whether the state should actively ask questions.
  - root: The beginning State.
  - relation: Relations between states. On the left is the certain output from one particular state, and on the right is the connected state which matches the output.
  - environment_type: Agent in different states do not share memory when competing, but share memory when coordinate is used.


Part 2: Examples
~~~~~~~~~~~~~~~~

Please refer to our Agents Demonstrations for more information. You can use them as reference.

Single-Agent Mode: 
----------------------------

Oculist Agent—Medical Use:
^^^^^^^^^^^^^^^^^^^^^^^^^

Model Description
~~~~~~~~~~~~~~~~~

The oculist agent acts as a consultant, providing professional advice and enabling online reservations for patients.

How to run our Raw Model
~~~~~~~~~~~~~~~~~~~~~~~~

- If you want to simply talk to our given Oculist agent, please run these codes:

  .. code-block:: bash

     cd examples
     python run.py --agent Single_Agent/Oculist_Agent/config.json

- If you want to run it in the gradient interface:

  .. code-block:: bash

     cd examples
     python run_gradio.py --agent Single_Agent/Oculist_Agent/config.json

- You can easily change the agent by changing the (--agent and --config) parameters.

- 🧠 If you want to generate other customized agents, please follow our instructions above.


SOP Demonstration:
~~~~~~~~~~~~~~~~~~

The SOP of our Oculist Agent is shown below:

[Image]

Explanations:

The SOP of the Oculist Agent consists of four states, each finishing their parts of the whole workflow.

- knowledge_base state: Provide expertised suggestions for patients, offering guidance to the hospital.
- book_card state: Send the information card for patients to fill in and offer reservation in advance.
- welcome_end state: Respond to other questions such as 'How can I get to the hospital?', 'When should I come?', etc.
- response_end state: Send particular messages, ending the whole conversation.

The typical JSON File of the Oculist Agent is shown as follows:

.. code-block:: json
{ 
  "config":{
    "API_KEY" : "API_KEY",
    "PROXY" : "PROXY",
    "MAX_CHAT_HISTORY" : "5",
    "MIN_CATEGORY_SIM" : "0.7",
    "FETSIZE" : "3",
    "User_Names" : "[\"Agod\"]",
    "Embed_Model" : "intfloat/multilingual-e5-large"
  },
  "LLM_type": "OpenAI",
  "LLM": {
    "temperature": 0.3,
    "model": "gpt-3.5-turbo-16k-0613",
    "log_path": "logs/god"
  },
  "root": "knowledge_response",
  "relations": {
    "knowledge_response": {
      "1": "knowledge_response_book_card",
      "0": "knowledge_response"
    },
    "knowledge_response_book_card": {
      "1": "end",
      "0": "knowledge_response_book_card"
    },
    "end": {
      "0": "knowledge_response_end"
    },
    "knowledge_response_end": {
      "0": "knowledge_response_end"
    }
  },
  "agents": {
    "Wu Jialong": {
      "style":"humorous",
      "roles":{
      "knowledge_response": "Oculist",
      "knowledge_response_book_card": "Oculist",
      "knowledge_response_end": "Oculist",
      "end": "Oculist"
      }
    },
    "Agod": {
      "style":"Cold and cunning",
      "roles":{
      "knowledge_response": "Customer",
      "knowledge_response_book_card": "Customer",
      "knowledge_response_end": "Customer",
      "end": "Customer"
      }
    }
  },
  "states": {
    "knowledge_response": {
      "name": "knowledge_response",
      "roles": [
        "Oculist",
        "Customer"
      ],
      "begin_role":"Oculist",
      "begin_query" :"Welcome to consult, do you have any questions?",
      "agent_states": {
        "Oculist": {
          "style": {
            "role": "Eye hospital customer service"
          },
          "task": {
            "task": "Guide the user to go to the hospital for an examination and answer questions related to my hospital."
          },
          "rule": {
            "rule": "Your language should be concise and avoid excessive words. You need to guide me repeatedly. When the user explicitly refuses to visit the hospital, inquire about their concerns and encourage them to come for consultation, such as: \"Do you have any concerns?\" or \"Our hospital has highly professional doctors who you can discuss with in person.\" When the user expresses doubts with responses like \"I'll think about it,\" \"I'll consider it,\" or \"I need to see more,\" introduce the advantages of the hospital and guide them to come for consultation. Remember, after responding to me, guide me to visit your hospital for an examination."
          },
          "KnowledgeBaseComponent": {
            "top_k": 1,
            "type": "QA",
            "knowledge_path": "Single_Agent/Oculist_Agent/database.json"
          }
        },
        "Customer":{
        }
      },
      "controller": {
        "controller_type":"order",
        "judge_system_prompt": "What you need to do now is determine whether the user agrees to go to the hospital. Based on the user's answer and combined with previous conversations, it is determined whether the user agrees to go to the hospital. \nIf the user agrees to go to the hospital, you need to return <end>1</end>, if not, you need to return <end>0</end>. \nYou need to pay special attention to what the Assistant and user said in the context. When the user answers OK, uh-huh, no more questions, etc., return <end>1</end>",
        "judge_last_prompt": "Please contact the above to extract <end> and </end>. Do not perform additional output. Please strictly follow the above format for output! Remember, please strictly follow the above format for output!",
        "judge_extract_words": "end"
      }
    },
    "knowledge_response_book_card": {
      "name": "knowledge_response_book_card",
      "roles": [
        "Oculist",
        "Customer"
      ],
      "agent_states": {
        "Oculist": {
          "style": {
            "role": "Eye hospital customer service"
          },
          "task": {
            "task": "Guide users to fill out appointment cards and answer hospital-related questions"
          },
          "rule": {
            "rule": "Your language should be as concise as possible, without too much nonsense. The copy of the invitation card is: Please copy and fill in the following information and send it to me to complete the reservation. \n[Name]:\n[Telephone]:\n[Your approximate location]: District Degree]: \n The preoperative examination process includes mydriasis. After mydriasis, your vision will be blurred for 4-6 hours, which affects driving safety, so please do not drive to the hospital by yourself, and arrange your personal itinerary after the examination. You need to repeatedly invite users to fill out invitation cards. When users are chatting, euphemistic replies guide users to fill in the appointment card, such as: \"I can't provide detailed information about your question. If you need to go to the hospital for eye consultation, I can make an appointment for you.\" When users have concerns, such as: Users reply with \"I want to think about it,\" \"I'll think about it,\" \"I want to see it again,\" etc., introducing the hospital's advantages and guiding users to fill in the appointment card. If the user does not fill in the phone number completely, the user will be reminded to add the phone number."
          },
          "KnowledgeBaseComponent": {
            "top_k": 1,
            "type": "QA",
            "knowledge_path": "Single_Agent/Oculist_Agent/database.json"
          }
        },
        "Customer":{
        }
      },
      "controller": {
        "controller_type":"order",
        "judge_system_prompt": "Based on the user's answer, analyze its relationship with the previous conversation and determine whether the user has filled out the appointment card. \n If the user fills in the phone information in the appointment card, output <end>1</end>\nIf the user does not fill in completely or the format is wrong, output <end>0</end>\n You need to pay special attention to the context ,Assitant and user said what respectively. When the user answers [Telephone]: 15563665210, <end>1</end> is returned. When the user answers [Telephone]: 15, <end>0</end> is returned because it is not filled in completely. When the user answers [Telephone]: abs, <end>0</end> is returned because it is not filled in completely.",
        "judge_last_prompt": "Please contact the above to extract <end> and </end>. Do not perform additional output. Please strictly follow the above format for output! Remember, please strictly follow the above format for output!",
        "judge_extract_words": "end"
      }
    },
    "knowledge_response_end": {
      "controller": {
        "controller_type":"order"
      },
      "name": "knowledge_response_end",
      "roles": [
        "Oculist",
        "Customer"
      ],
      "agent_states": {
        "Oculist": {
          "style": {
            "role": "Eye hospital customer service"
          },
          "task": {
            "task": "Answer relevant questions from users."
          },
          "rule": {
            "rule": "Your language should be as concise as possible and don’t talk too much."
          },
          "KnowledgeBaseComponent": {
            "top_k": 1,
            "type": "QA",
            "knowledge_path": "Single_Agent/Oculist_Agent/database.json"
          }
        },
        "Customer":{
        }
      }
    },
    "end": {
      "name": "end",
      "roles": [
        "Oculist",
        "Customer"
      ],
      "agent_states": {
        "controller": {
          "controller_type":"order"
        },
        "Oculist": {
          "StaticComponent": {
            "output": "我会帮您预约好名额，请您合理安排好时间。届时我会在二楼眼科分诊台等您。"
          }
        },
        "Customer":{
        }
      }
    }
  }
}


If you want to learn more about our JSON File or review the JSON file-generating process, please refer to our instructions.

Other than the Oculist Agent, we also provide various types of Agents, which can be seen in our AgentHub part:
💬Yang Bufan—Chatting Bot: 
~~~~~~~~~~~~~~~~~~~~~~~~

📋Youcai Agent—Policy Consultant:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

🏢Zhaoshang Agent—Commercial Assistant:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

🤖🤖Multi-Agent Mode: 
-------------------------------

📚Fiction Studio--Step-by-step fiction generating:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Model Description
~~~~~~~~~~~~~~~~~

The fiction studio is a typical example of the Multi-Agent Mode. Several writers work together to create a particular type of novel. By deciding and writing the abstract at first, and sequently adding details and scripts, a long novel can be easily generated. During the whole process, several writers are applied to offer advice and modify certain contents.

How to run our Raw Model
~~~~~~~~~~~~~~~~~~~~~~~~

If you want to simply run our Fiction Studio Mode, please run these codes:

.. code::

   cd examples
   python run_cmd.py --agent fiction.json

If you want to generate other customized agents, please follow our instructions above.

SOP Demonstration:
~~~~~~~~~~~~~~~~~~

The SOP of our Fiction Studio Mode is shown below:

[Image]

Explanations:
  The SOP of the Fiction Studio Mode consists of two Nodes, each containing one certain part of the whole workflow.
  - Node 1: Is responsible for generating an initial outline based on the given novel style, theme, etc., and suggestions for improvement are provided by the Outline Adviser.
  - Node 2: Is responsible for expanding upon the preliminary outline, adding suitable content, and incorporating relevant details.

The typical JSON File of Fiction Studio Mode is shown as follows:

.. code:: json

   {
    "temperature": 0.3,
    "active_mode": true,
    "log_path": "./",
    "environment_prompt": "现在需要写一本关于古代穿越剧的剧本，剧本大概需要有5个章节。",
    "nodes": {
        "Node 1": {
            "name": "Node 1",
            "agent_states": {
                "大纲写作者1": {
                    "style": {
                        "name": "小亮",
                        "role": "中文写作大师，拥有丰富的创作经验，擅长写大纲",
                        "style": "用清晰、简洁的语言，突出关键信息，避免过度描述，以便与另一个作家小刚高效沟通。"
                    },
                    "task": {
                        "task": "你是小亮，负责在与另一个作家小刚合作的情况下，共同创作一个小说大纲。你应该在创作过程中积极地提供想法、人物背景和情节线索。"
                    },
                    "rule": {
                        "rule": "你需要首先确定人物和章节目录，然后丰富章节。人物包括性别、姓名、工作、性格、讲话风格、背景以及和其他人的关系，每个章节的大概情节应包括关键事件、人物发展和情感转折。人物特性和背景应该能够支持情节的发展，同时为整个故事增加深度。"
                    },
                    "demonstration": {
                        "demonstration": "# 人物\n## 人物1：\n- 性别：男\n- 姓名：李安迪\n- 工作：互联网公司程序员\n- 性格：以自我为中心，情绪起伏大、缺乏责任感和成熟度\n- 讲话风格：充满情绪化、攻击性、容易激动，经常使用威胁、责骂、挖苦或讽刺的言辞\n- 和其他人的关系：程雨婷的丈夫，二者育有一个儿子李力\n- 背景：平时工作很忙，最近刚完成一个很繁重的项目\n\n## 人物2:\n- 性别：女\n- 姓名：程雨婷\n- 工作：高中语文老师\n- 性格：固执强势，但关心家人，有大局观\n- 讲话风格：直接坚定，直接表达自己的观点，不会拐弯抹角；强势自信，会在说话中展现出自信和权威；语气坚决\n- 和其他人的关系：李安迪的妻子，二者育有一个儿子李力\n- 背景：平时李安迪工作忙，而你工作相对轻松，大部分的育儿工作由你来完成。你很尊重自己的父母，不愿意因为自己家的事情麻烦他们\n\n# 大纲\n## 章节1\n- 标题：意外的邂逅\n- 主要内容：23岁刚毕业的大学生李安迪进入了上海科技公司，成为程序员，工作十分上进。同样为23岁的大学生程雨婷，也在......"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲写作者1小亮，只用代表大纲写作者1小亮进行回答，输出格式为大纲写作者1（小亮）：...."
                    },
                    "config": ["style", "task", "rule", "demonstration", "last"]
                },
                "大纲写作者2": {
                    "style": {
                        "name": "小刚",
                        "role": "中文写作大师，拥有丰富的创作经验和编剧撰写经验，擅长对大纲进行扩写",
                        "style": "使用富有想象力的语言，注重情感和细节的描绘，以激发创作灵感，同时能够理解和回应作家小亮的意见。"
                    },
                    "task": {
                        "task": "你是小刚，你需要和另外一个作家小亮合作，共同构思小说大纲。你需要积极参与创意讨论，提供新颖的想法，确保人物和情节的连贯性。"
                    },
                    "rule": {
                        "rule": "你需要首先确定人物和章节目录，然后丰富章节。人物包括性别、姓名、工作、性格、讲话风格、背景以及和其他人的关系。每章情节的构思应该与整体题材紧密相连，确保情节逻辑流畅，人物形象栩栩如生。在提出人物特性和背景时，请考虑它们如何促进故事的进展。"
                    },
                    "demonstration": {
                        "demonstration": "# 人物\n## 人物1：\n- 性别：男\n- 姓名：李安迪\n- 工作：互联网公司程序员\n- 性格：以自我为中心，情绪起伏大、缺乏责任感和成熟度\n- 讲话风格：充满情绪化、攻击性、容易激动，经常使用威胁、责骂、挖苦或讽刺的言辞\n- 和其他人的关系：程雨婷的丈夫，二者育有一个儿子李力\n- 背景：平时工作很忙，最近刚完成一个很繁重的项目\n\n## 人物2:\n- 性别：女\n- 姓名：程雨婷\n- 工作：高中语文老师\n- 性格：固执强势，但关心家人，有大局观\n- 讲话风格：直接坚定，直接表达自己的观点，不会拐弯抹角；强势自信，会在说话中展现出自信和权威；语气坚决\n- 和其他人的关系：李安迪的妻子，二者育有一个儿子李力\n- 背景：平时李安迪工作忙，而你工作相对轻松，大部分的育儿工作由你来完成。你很尊重自己的父母，不愿意因为自己家的事情麻烦他们\n\n# 大纲\n## 章节1\n- 标题：意外的邂逅\n- 主要内容：23岁刚毕业的大学生李安迪进入了上海科技公司，成为程序员，工作十分上进。同样为23岁的大学生程雨婷，也在......"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲写作者2小刚，只用代表大纲写作者2小刚进行回答，输出格式为大纲写作者2（小刚）：...."
                    },
                    "config": ["style", "task", "rule", "demonstration", "last"]
                },
                "大纲建议者": {
                    "style": {
                        "name": "小风",
                        "role": "影视编剧创作者，擅长将经典的小说改编成剧本进行演绎，拥有丰富的修改大纲和提供修改意见的经历",
                        "style": "专业、友好、精简的语言，指出潜在问题、改进机会以及对情节和人物的建议，以协助作家们进一步完善创意。"
                    },
                    "task": {
                        "task": "你是小风，你的职责是根据作家小刚和小亮提供的大纲，进行内容审查和意见提供。请务必需要确保故事的内在逻辑、一致性和吸引力。"
                    },
                    "rule": {
                        "rule": "你应关注故事的整体结构，确保每个章节之间的过渡平滑，人物行为和动机合理。编辑可以提供关于情节深度、紧凑性和情感共鸣的建议，同时保留作家们的创作风格。"
                    },
                    "demonstration": {
                        "demonstration": "# 建议1：\n- 问题：目前设置的人物还不够多，内容情节不够丰富\n- 修改意见：建议额外增加三个不同的人物，来丰富情节\n\n# 建议2：\n- 问题：小亮对于人物2的塑造要比小刚对于任务2的塑造更好，而人物1是小刚塑造的更高\n- 修改意见：建议人物1采用小刚的结果，人物2采用小亮的结果。"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲建议者小风，只用代表大纲建议者小风进行回答，输出格式为大纲建议者（小风）：...."
                    },
                    "config": ["style", "task", "rule", "demonstration", "last"]
                }
            },
            "controller": {
                "judge_system_prompt": "判断当前的大纲是否按照要求完成，如果完成的话输出<结束>1</结束>，否则输出<结束>0</结束>",
                "judge_last_prompt": "判断当前的大纲是否按照要求完成，如果完成的话输出<结束>1</结束>，否则输出<结束>0</结束>",
                "judge_extract_words": "结束",
                "call_system_prompt": "目前有3个人进行分工合作来完成关于小说大纲的生成，他们分别为大纲写作者1（小亮），大纲写作者2（小刚），大纲建议者（小风）。。根据他们的对话，你需要判断下一个是谁来发言。",
                "call_last_prompt": "根据当前的对话，判断下一个是谁来发言。如果是大纲写作者1（小亮），则输出<结束>大纲写作者1</结束>。如果是大纲写作者2（小刚），则输出<结束>大纲写作者2</结束>。如果是大纲建议者（小风），则输出<结束>大纲建议者</结束>",
                "call_extract_words": "结束"
            },
            "root": true,
            "is_interactive": true
        },
        "Node 2": {
            "name": "Node 2",
            "agent_states": {
                "大纲扩写者1": {
                    "style": {
                        "name": "小明",
                        "role": "中文写作大师，拥有丰富的创作经验，擅长以大纲为基础进行扩写",
                        "style": "用生动的、富有情感的语言，让读者能够沉浸在故事中。与作家小明密切合作，交流创意和解决情节问题。"
                    },
                    "task": {
                        "task": "你是小明，需要负责与作家小白共同将大纲转化为具体的章节内容。你需要在每个章节中添加详细的情节，以及扩展人物关系和发展，重点关注情节的起因和结果，确保一致。"
                    },
                    "rule": {
                        "rule": "每个章节的内容应紧密遵循大纲，确保情节的延续和连贯。人物行为和对话应当与之前设定的特性和背景保持一致。"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲扩写者1小明，只用代表大纲扩写者1小明进行回答，输出格式为大纲扩写者1（小明）：...."
                    },
                    "config": ["style", "task", "rule", "last"]
                },
                "大纲扩写者2": {
                    "style": {
                        "name": "小白",
                        "role": "中文写作大师，拥有丰富的创作经验和编剧撰写经验，擅长以大纲为基础进行扩写",
                        "style": "使用引人入胜的描写和令人难以忘怀的情节，与作家小明共同构建丰富的故事世界。"
                    },
                    "task": {
                        "task": "你是小白，你需要与小白协同努力，将大纲细化为具体的章节。你需要提供深入的背景描述、丰富的情感体验，重点关注情节的起因和结果，确保一致。"
                    },
                    "rule": {
                        "rule": "应在扩写过程中保持大纲的核心情节，同时可以适度地拓展细节，使故事更具深度和张力。"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲扩写者2小白，只用代表大纲扩写者2小白进行回答，输出格式为大纲扩写者2（小白）：...."
                    },
                    "config": ["style", "task", "rule", "last"]
                },
                "大纲扩写建议者": {
                    "style": {
                        "name": "小红",
                        "role": "影视编剧创作者，擅长将经典的小说改编成剧本进行演绎，拥有丰富的修改大纲和提供修改意见的经历",
                        "style": "专业、友好、精简的语言，指出章节中的潜在问题、改进机会和对情节的建议，以协助作家小明和小白进一步完善创作。"
                    },
                    "task": {
                        "task": "你是小红，需要审阅作家小明和小白的章节内容，确保情节逻辑、连贯性和整体质量，此外需要注意故事结构、人物塑造和情感共鸣，重点关注起因和结果，并提供有针对性的建议。"
                    },
                    "rule": {
                        "rule": "你需要关注章节之间的过渡，确保情节的内在逻辑，人物行为的合理性，以及情感体验的真实性。你可以提供有关情节深度、对话自然性和紧凑性的建议，同时保留作家们的创作风格。"
                    },
                    "last": {
                        "last_prompt": "切记，你的身份是大纲扩写建议者小红，只用代表大纲扩写建议者小红进行回答，输出格式为大纲扩写建议者（小红）：...."
                    },
                    "config": ["style", "task", "rule", "last"]
                }
            },
            "controller": {
                "judge_system_prompt": "判断当前的大纲是否扩写完成，如果完成的话输出<结束>1</结束>，否则输出<结束>0</结束>",
                "judge_last_prompt": "根据上面的回答判断大纲是否已经扩写完成，如果完成的话输出<{EXTRACT_PROMPT_TEMPLATE}>1</{EXTRACT_PROMPT_TEMPLATE}>，否则输出<{EXTRACT_PROMPT_TEMPLATE}>0</{EXTRACT_PROMPT_TEMPLATE}>",
                "judge_extract_words": "结束",
                "call_system_prompt": "目前有3个人进行分工合作来对大纲进行扩写，他们分别为大纲扩写者1（小明），大纲扩写者2（小白），大纲扩写建议者（小红）。。根据他们的对话，你需要判断下一个是谁来发言。",
                "call_last_prompt": "根据当前的对话，判断下一个是谁来发言。如果是大纲扩写者1（小明），则输出<结束>大纲扩写者1</结束>。如果是大纲扩写者2（小白），则输出<结束>大纲扩写者2</结束>。如果是大纲扩写建议者（小红），则输出<结束>大纲扩写建议者</结束>",
                "call_extract_words": "结束"
            },
            "root": false,
            "is_interactive": true
        }
    },
    "relation": {
        "Node 1": {
            "0": "Node 1",
            "1": "Node 2"
        },
        "Node 2": {
            "0": "Node 2"
        }
    }
   }

If you want to learn more about our JSON File or review the JSON file-generating process, please refer to our instructions.

