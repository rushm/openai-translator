【gradio_server.py修改内容】
/*
def translation(input_file, source_language, target_language, translation_style):
    LOG.debug(f"[翻译任务]\n源文件: {input_file.name}\n源语言: {source_language}\n目标语言: {target_language}\n翻译风格：{translation_style}")

    output_file_path = Translator.translate_pdf(
        input_file.name, source_language=source_language, target_language=target_language, translation_style=translation_style)

    return output_file_path

def launch_gradio():

    iface = gr.Interface(
        fn=translation,
        title="OpenAI-Translator v2.0(PDF 电子书翻译工具)",
        inputs=[
            gr.File(label="上传PDF文件"),
            gr.Textbox(label="源语言（默认：英文）", placeholder="English", value="English"),
            gr.Textbox(label="目标语言（默认：中文）", placeholder="Chinese", value="Chinese"),
            gr.Textbox(label="翻译风格（默认：小说）", placeholder="novel", value="novel")
        ],
        outputs=[
            gr.File(label="下载翻译文件")
        ],
        allow_flagging="never"
    )
*/ translation函数添加 translation_style 翻译风格参数，launch 函数添加翻译风格标签，默认为小说

【pdf_translator.py修改内容】
/*
def translate_pdf(self,
                    input_file: str,
                    output_file_format: str = 'markdown',
                    source_language: str = "English",
                    target_language: str = 'Chinese',
                    translation_style: str = 'novel',
                    pages: Optional[int] = None):
        
        self.book = self.pdf_parser.parse_pdf(input_file, pages)

        for page_idx, page in enumerate(self.book.pages):
            for content_idx, content in enumerate(page.contents):
                # Translate content.original
                translation, status = self.translate_chain.run(content, source_language, target_language, translation_style)
                # Update the content in self.book.pages directly
                self.book.pages[page_idx].contents[content_idx].set_translation(translation, status)
        
        return self.writer.save_translated_book(self.book, output_file_format)
*/ translate_pdf函数添加 translation_style 翻译风格参数,默认为小说

【translation_chain.py修改内容】
/*
class TranslationChain:
    def __init__(self, model_name: str = "gpt-3.5-turbo", verbose: bool = True):
        
        # 翻译任务指令始终由 System 角色承担
        template = (
            """You are a translation expert, proficient in various languages and support stylized translation. \n
            Translates {source_language} to {target_language},the translation style is {translation_style}."""
        )
        system_message_prompt = SystemMessagePromptTemplate.from_template(template)

        # 待翻译文本由 Human 角色输入
        human_template = "{text}"
        human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)

        # 使用 System 和 Human 角色的提示模板构造 ChatPromptTemplate
        chat_prompt_template = ChatPromptTemplate.from_messages(
            [system_message_prompt, human_message_prompt]
        )

        # 为了翻译结果的稳定性，将 temperature 设置为 0
        chat = ChatOpenAI(model_name=model_name, temperature=0, verbose=verbose)

        self.chain = LLMChain(llm=chat, prompt=chat_prompt_template, verbose=verbose)

    def run(self, text: str, source_language: str, target_language: str, translation_style: str) -> (str, bool):
        result = ""
        try:
            result = self.chain.run({
                "text": text,
                "source_language": source_language,
                "target_language": target_language,
                "translation_style": translation_style,
            })
        except Exception as e:
            LOG.error(f"An error occurred during translation: {e}")
            return result, False

        return result, True
*/ template添加翻译风格变量{translation_style}，run方法添加translation_style参数
