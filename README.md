# terminal_to_clipboard
Copies terminal history to clipboard from the last time "clear" was typed. Helpful for GPT debugging.
 
Just copy this command into terminal. 
Type "clear" and do some commands.
Type "clip" and everything from the "last typed clear" command will be in your clipboard.

The install command just adds this to .bashrc.

```
#!/bin/bash

# Step 1: Add the correct 'clip' function to ~/.bashrc
echo "Adding 'clip' function to ~/.bashrc..."

cat << 'EOF' >> ~/.bashrc

# Function to capture terminal output after the most recent 'clear'
clip() {
    # Save the current terminal history to a temporary file
    history > ~/full_terminal_history.txt

    # Find the last occurrence of 'clear' in the terminal history
    last_clear_line=$(grep -n "clear" ~/full_terminal_history.txt | tail -1 | cut -d: -f1)

    # If 'clear' was found, grab everything after it, otherwise take the entire history
    if [ -n "$last_clear_line" ]; then
        tail -n +$((last_clear_line + 1)) ~/full_terminal_history.txt > ~/filtered_terminal_history.txt
    else
        cat ~/full_terminal_history.txt > ~/filtered_terminal_history.txt
    fi

    # Remove line numbers and exclude 'clip' commands from the filtered history
    sed '/clip/d' ~/filtered_terminal_history.txt | sed 's/^[[:space:]]*[0-9]*[[:space:]]*//' > ~/final_terminal_history.txt

    # Copy the cleaned terminal history to the clipboard
    cat ~/final_terminal_history.txt | xclip -selection clipboard

    # Provide feedback
    echo "Terminal output after last 'clear' copied to clipboard."
}

EOF

# Step 2: Source the .bashrc to apply changes
echo "Sourcing ~/.bashrc to apply changes..."
source ~/.bashrc

echo "Installation complete! You can now use the 'clip' command."
```
