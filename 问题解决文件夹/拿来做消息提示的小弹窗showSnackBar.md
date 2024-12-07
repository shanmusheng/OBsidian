
## showSnackBar
```
void _showSnackBar(BuildContext context, String message) {  
  ScaffoldMessenger.of(context).showSnackBar(  
    SnackBar(content: Text(message)),  
  );  
}
```
