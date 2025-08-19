# elastic-search-chatbot

Creating an Elasticsearch-backed chatbot involves several steps, including setting up an Elasticsearch server, indexing your data, and writing the Python code for the chatbot. Below is a simple example of a Python program using the `elasticsearch` package to build an Elasticsearch-backed chatbot. This example assumes you have a working Elasticsearch server and some data indexed.

```python
from elasticsearch import Elasticsearch, exceptions
from elasticsearch.helpers import bulk
import json

class ElasticSearchChatbot:
    def __init__(self, host='localhost', port=9200, index_name='chatbot-index'):
        # Initialize Elasticsearch client
        self.es = Elasticsearch([{'host': host, 'port': port}])
        self.index_name = index_name
        self.init_index()

    def init_index(self):
        # Create an index if it doesn't exist
        if not self.es.indices.exists(index=self.index_name):
            self.es.indices.create(index=self.index_name)
            print(f"Created index: {self.index_name}")
        else:
            print(f"Index already exists: {self.index_name}")

    def index_data(self, data_file):
        # Bulk index data into Elasticsearch from a JSON file
        try:
            with open(data_file, 'r') as file:
                data = json.load(file)
            actions = [
                {
                    "_index": self.index_name,
                    "_source": item
                }
                for item in data
            ]
            success, _ = bulk(self.es, actions)
            print(f"Successfully indexed {success} documents.")
        except FileNotFoundError:
            print("Data file not found. Please check the file path.")
        except json.JSONDecodeError:
            print("Error decoding JSON from data file.")
        except exceptions.ElasticsearchException as e:
            print(f"Elasticsearch error: {e}")

    def search(self, query):
        # Search the indexed data
        try:
            response = self.es.search(
                index=self.index_name,
                body={
                    "query": {
                        "match": {
                            "message": query
                        }
                    }
                }
            )
            return self._parse_response(response)
        except exceptions.ElasticsearchException as e:
            print(f"Elasticsearch error: {e}")
            return None

    def _parse_response(self, response):
        # Parse the response from Elasticsearch
        try:
            hits = response['hits']['hits']
            if not hits:
                print("No relevant answers found.")
                return []

            for hit in hits:
                source = hit['_source']
                print(f"Answer: {source.get('answer', 'Not available')}")

            return [hit['_source'] for hit in hits]
        
        except KeyError:
            print("Unexpected response format.")
            return []

def main():
    # Initialize the Elasticsearch-backed chatbot
    chatbot = ElasticSearchChatbot()
    
    # Index data if needed (ensure you have a data.json file or similar structured JSON file)
    # Uncomment the line below after placing your data file path and running it one-time to index data.
    # chatbot.index_data('path/to/your/data.json')
    
    # Example interaction loop with the chatbot
    print("Welcome to the ElasticSearch Chatbot! Type 'exit' to end the chat.")
    while True:
        user_input = input("You: ")
        if user_input.lower() == 'exit':
            print("Goodbye!")
            break
        
        response = chatbot.search(user_input)
        if response:
            for answer in response:
                print(f"Chatbot: {answer.get('answer', 'No suitable response found')}")
        else:
            print("Chatbot: Sorry, I couldn't find an answer to your question.")

if __name__ == "__main__":
    main()
```

### Key Points:
- **Initialization**: Initialize Elasticsearch client and create an index if it doesn't exist.
- **Data Indexing**: Use `bulk` helper to index data from a JSON file. The JSON should be structured such that each entry has at least a `message` and an `answer` field.
- **Error Handling**: Handle potential errors such as missing files, JSON decode errors, and Elasticsearch-specific exceptions.
- **Search Functionality**: Use Elasticsearch’s search capabilities to find relevant documents based on a simple `match` query.
- **Interaction**: Provides a simple text-based interface for chatting.

### Required Python Package
Install necessary packages before running the script:
```bash
pip install elasticsearch
```

### Data Format
Ensure your `data.json` file (or whichever format you're using) has entries like:
```json
[
    {"message": "How to reset password?", "answer": "You can reset your password by going to the settings page."},
    {"message": "What is my account balance?", "answer": "Your account balance can be viewed in your dashboard."}
]
```

This code is a prototype and can be extended for more complex logic, including natural language processing, improved match algorithms, and more detailed responses.