import tkinter as tk
from tkinter import messagebox


class Theme:
    def __init__(self) -> None:
        self.bg = "#101418"
        self.panel = "#171c22"
        self.text = "#e9edf1"
        self.muted = "#9aa6b2"
        self.accent = "#4cc9f0"
        self.accent_soft = "#2a3742"
        self.tile = "#1f2630"
        self.tile_hover = "#2a3440"
        self.tile_disabled = "#151a20"
        self.win = "#3a6f55"
        self.reset = "#384454"
        self.quit = "#4b2f2f"
        self.shadow = "#0b0e12"


class TicTacToeUI:
    def __init__(self, root: tk.Tk) -> None:
        self.root = root
        self.theme = Theme()
        self.root.title("Tic Tac Toe")
        self.root.resizable(False, False)
        self.root.configure(bg=self.theme.bg)

        self.current_player = "X"
        self.board = [["" for _ in range(3)] for _ in range(3)]
        self.buttons: list[list[tk.Button]] = []

        self.status_var = tk.StringVar(value="Player X's turn")

        header = tk.Frame(self.root, padx=16, pady=16, bg=self.theme.bg)
        header.pack(fill="x")

        title = tk.Label(
            header,
            text="TIC TAC TOE",
            font=("Segoe UI", 16, "bold"),
            fg=self.theme.text,
            bg=self.theme.bg,
        )
        title.pack(side="left")

        status = tk.Label(
            header,
            textvariable=self.status_var,
            font=("Segoe UI", 11, "bold"),
            fg=self.theme.accent,
            bg=self.theme.bg,
            padx=12,
            pady=4,
        )
        status.pack(side="right")

        board_frame = tk.Frame(self.root, padx=16, pady=16, bg=self.theme.panel)
        board_frame.pack()

        for row in range(3):
            button_row: list[tk.Button] = []
            for col in range(3):
                button = tk.Button(
                    board_frame,
                    text="",
                    width=5,
                    height=2,
                    font=("Segoe UI", 28, "bold"),
                    relief="flat",
                    bg=self.theme.tile,
                    fg=self.theme.text,
                    activebackground=self.theme.tile_hover,
                    activeforeground=self.theme.text,
                    highlightthickness=0,
                    bd=0,
                    command=lambda r=row, c=col: self.handle_move(r, c),
                )
                button.grid(row=row, column=col, padx=8, pady=8)
                button.bind("<Enter>", lambda e, b=button: self.on_hover(b, True))
                button.bind("<Leave>", lambda e, b=button: self.on_hover(b, False))
                button_row.append(button)
            self.buttons.append(button_row)

        footer = tk.Frame(self.root, padx=16, pady=16, bg=self.theme.bg)
        footer.pack(fill="x")

        reset_btn = tk.Button(
            footer,
            text="Reset",
            font=("Segoe UI", 10, "bold"),
            fg=self.theme.text,
            bg=self.theme.reset,
            activebackground=self.theme.tile_hover,
            activeforeground=self.theme.text,
            relief="flat",
            padx=16,
            pady=6,
            command=self.reset_game,
        )
        reset_btn.pack(side="left")

        quit_btn = tk.Button(
            footer,
            text="Quit",
            font=("Segoe UI", 10, "bold"),
            fg=self.theme.text,
            bg=self.theme.quit,
            activebackground="#5a3a3a",
            activeforeground=self.theme.text,
            relief="flat",
            padx=16,
            pady=6,
            command=self.root.destroy,
        )
        quit_btn.pack(side="right")

    def on_hover(self, button: tk.Button, is_hover: bool) -> None:
        if button["state"] == "disabled":
            return
        button.configure(bg=self.theme.tile_hover if is_hover else self.theme.tile)

    def handle_move(self, row: int, col: int) -> None:
        if self.board[row][col] != "":
            return

        self.board[row][col] = self.current_player
        symbol_color = self.theme.accent if self.current_player == "X" else self.theme.text
        self.buttons[row][col].config(
            text=self.current_player,
            state="disabled",
            bg=self.theme.tile_disabled,
            fg=symbol_color,
        )

        winner = self.check_winner()
        if winner:
            self.highlight_winner(winner)
            messagebox.showinfo("Game Over", f"Player {winner} wins!")
            self.disable_board()
            return

        if self.is_draw():
            messagebox.showinfo("Game Over", "It's a draw!")
            self.disable_board()
            return

        self.toggle_player()

    def toggle_player(self) -> None:
        self.current_player = "O" if self.current_player == "X" else "X"
        self.status_var.set(f"Player {self.current_player}'s turn")

    def check_winner(self) -> str | None:
        lines = []
        lines.extend(self.board)
        lines.extend([[self.board[r][c] for r in range(3)] for c in range(3)])
        lines.append([self.board[i][i] for i in range(3)])
        lines.append([self.board[i][2 - i] for i in range(3)])

        for line in lines:
            if line[0] != "" and line.count(line[0]) == 3:
                return line[0]
        return None

    def highlight_winner(self, winner: str) -> None:
        winning_positions = []

        for r in range(3):
            if self.board[r][0] == self.board[r][1] == self.board[r][2] == winner:
                winning_positions = [(r, 0), (r, 1), (r, 2)]

        for c in range(3):
            if self.board[0][c] == self.board[1][c] == self.board[2][c] == winner:
                winning_positions = [(0, c), (1, c), (2, c)]

        if self.board[0][0] == self.board[1][1] == self.board[2][2] == winner:
            winning_positions = [(0, 0), (1, 1), (2, 2)]

        if self.board[0][2] == self.board[1][1] == self.board[2][0] == winner:
            winning_positions = [(0, 2), (1, 1), (2, 0)]

        for r, c in winning_positions:
            self.buttons[r][c].config(bg=self.theme.win, fg=self.theme.text)

    def is_draw(self) -> bool:
        return all(self.board[r][c] != "" for r in range(3) for c in range(3))

    def disable_board(self) -> None:
        for row in self.buttons:
            for btn in row:
                btn.config(state="disabled")

    def reset_game(self) -> None:
        self.current_player = "X"
        self.board = [["" for _ in range(3)] for _ in range(3)]
        self.status_var.set("Player X's turn")

        for row in self.buttons:
            for btn in row:
                btn.config(text="", state="normal", bg=self.theme.tile, fg=self.theme.text)


def main() -> None:
    root = tk.Tk()
    TicTacToeUI(root)
    root.mainloop()


if __name__ == "__main__":
    main()
