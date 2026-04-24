# Vector Database

The Core Problem: Traditional database like SQL are built for exact keyword matching and struggle with the meaning behind user query.
Vector database solve this by semantic meaning based matching

What are Vectors ? Machine learning models convert data (text,images) into list of numbers called vectors or embeddings.Thease text in mathematical space where similar meanings are positioned closed together,Allowing AI to understand relationship between concepts

RAG (Retrieval-Augmented Generations)=Retrieval-Augmented Generation (RAG) is a technique that enhances LLM accuracy by retrieving relevant data from external, trusted sources before generating a response.

Chunking Strategy:Line-by-line chunking is a method of dividing text for RAG (Retrieval-Augmented Generation) applications, where each line or sentence in a document is treated as an individual chunk. This strategy is particularly useful for structured, high-density data such as logs, dialogues, and transcripts, where each line conveys a specific, self-contained piece of information. 

Vector Search is used widely in Recommendations engine such as Spotify,Netflix, visual search (Pinterest), and anomaly detection in cybersecurity.


Example

```python
from pydantic import BaseModel, Field
from langchain.agents import create_agent


class ContactInfo(BaseModel):
    """Contact information for a person."""
    name: str = Field(description="The name of the person")
    email: str = Field(description="The email address of the person")
    phone: str = Field(description="The phone number of the person")

agent = create_agent(
    model="gpt-5",
    response_format=ContactInfo  # Auto-selects ProviderStrategy
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Extract contact info from: John Doe, john@example.com, (555) 123-4567"}]
})

print(result["structured_response"])
# ContactInfo(name='John Doe', email='john@example.com', phone='(555) 123-4567')
```