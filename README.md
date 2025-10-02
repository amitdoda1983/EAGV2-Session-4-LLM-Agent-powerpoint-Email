The MCP server has tools to do maths , update power point and send emails.

@mcp.tool()
async def create_live_powerpoint_slide(
    x1: float,
    y1: float,
    x2: float,
    y2: float,
    text: str = "Hello World"
) -> str:
    """
    Create a live PowerPoint slide with a rectangle from (x1, y1) to (x2, y2) and text inside.
    Coordinates are in points.
    """
    try:
        # Start PowerPoint
        ppt_app = win32com.client.Dispatch("PowerPoint.Application")
        ppt_app.Visible = True
        await asyncio.sleep(1)  # let PowerPoint initialize

        # Add new presentation
        presentation = ppt_app.Presentations.Add()
        await asyncio.sleep(1)

        # Blank slide layout = 12
        slide_layout = 12
        slide_count = len(presentation.Slides)
        slide_index = slide_count + 1
        slide = presentation.Slides.Add(slide_index, slide_layout)
        await asyncio.sleep(1)

        # Calculate width and height from coordinates
        rect_width = abs(x2 - x1)
        rect_height = abs(y2 - y1)
        left = min(x1, x2)
        top = min(y1, y2)

        # Add rectangle (msoShapeRectangle = 1)
        shape = slide.Shapes.AddShape(1, left, top, rect_width, rect_height)
        await asyncio.sleep(1)  # let shape render

        # Add text AFTER shape exists
        shape.TextFrame.TextRange.Text = text
        shape.TextFrame.TextRange.Font.Size = 32
        await asyncio.sleep(1)

        # Center text horizontally and vertically
        shape.TextFrame.TextRange.ParagraphFormat.Alignment = 2  # ppAlignCenter
        shape.TextFrame.VerticalAnchor = 3  # msoAnchorMiddle
        await asyncio.sleep(1)

        print(f"Rectangle drawn from ({x1},{y1}) to ({x2},{y2}) with text: '{text}'")
        return 'text added to power point'
    except Exception as e:
        print(f"Error: {e}")
        return 'Error'


        @mcp.tool()
async def send_email(to: str, subject: str, body: str) -> str:
    try:
        msg = MIMEText(body)
        msg["Subject"] = subject
        msg["From"] = GMAIL_USER
        msg["To"] = to
        
        with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
            server.login(GMAIL_USER, GMAIL_PASS)
            server.sendmail(GMAIL_USER, [to], msg.as_string())

        return f"Email successfully sent to {to}"
    except Exception as e:
        return f"Error sending email: {str(e)}"

The MCP client has the LLM system prompt definition and tool calling recommended by LLM Gemini.

Updated :
You must respond with EXACTLY ONE line in one of these formats (no additional text):
For function calls:
   FUNCTION_CALL: function_name|param1|param2|...
   
Important:
- When a function returns multiple values, you need to process all of them
- Do not repeat function calls with the same parameters
- once you have the final answer you have to send the result via email to a default email_id: amit.doda1983@gmail.com
- And also write it in powerpoint. with arguments "x1": 200, "y1": 130, "x2": 600, "y2": 430, "text": Final_answer

Examples:
- FUNCTION_CALL: add|5|3
- FUNCTION_CALL: strings_to_chars_to_int|INDIA
- FUNCTION_CALL: create_live_powerpoint_slide|200|130|600|430|FINAL_ANSWER: [42]
- FUNCTION_CALL: send_results_email|to|subject|FINAL_ANSWER: [42]
- FINAL_ANSWER: [42]


DO NOT include any explanations or additional text.
Your entire response should be a single line starting with either FUNCTION_CALL: or FINAL_ANSWER:"""
