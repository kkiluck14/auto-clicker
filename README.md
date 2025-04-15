import tkinter as tk
import pyautogui
import threading
import time
import json
import os

SAVE_FILE = "coords.json"

class AutoClickerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("3포인트 오토 클릭기")
        self.running = False
        self.entries = []

        for i in range(3):
            frame = tk.Frame(root)
            frame.pack(pady=5)
            tk.Label(frame, text=f"Point {i+1} (x, y):").pack(side=tk.LEFT)
            x_entry = tk.Entry(frame, width=5)
            x_entry.pack(side=tk.LEFT)
            y_entry = tk.Entry(frame, width=5)
            y_entry.pack(side=tk.LEFT)
            self.entries.append((x_entry, y_entry))

        self.start_btn = tk.Button(root, text="시작", command=self.start_clicking)
        self.start_btn.pack(pady=5)

        self.stop_btn = tk.Button(root, text="정지", command=self.stop_clicking, state=tk.DISABLED)
        self.stop_btn.pack(pady=5)

        self.save_btn = tk.Button(root, text="좌표 저장", command=self.save_coords)
        self.save_btn.pack(pady=5)

        self.status_label = tk.Label(root, text="대기 중")
        self.status_label.pack(pady=5)

        self.load_coords()

    def save_coords(self):
        try:
            coords = []
            for x_entry, y_entry in self.entries:
                x = int(x_entry.get())
                y = int(y_entry.get())
                coords.append({"x": x, "y": y})
            with open(SAVE_FILE, "w") as f:
                json.dump(coords, f)
            self.status_label.config(text="좌표 저장 완료")
        except ValueError:
            self.status_label.config(text="좌표를 올바르게 입력하세요.")

    def load_coords(self):
        if os.path.exists(SAVE_FILE):
            with open(SAVE_FILE, "r") as f:
                try:
                    coords = json.load(f)
                    for i, (x_entry, y_entry) in enumerate(self.entries):
                        x_entry.delete(0, tk.END)
                        y_entry.delete(0, tk.END)
                        x_entry.insert(0, coords[i]["x"])
                        y_entry.insert(0, coords[i]["y"])
                    self.status_label.config(text="저장된 좌표 로드 완료")
                except:
                    self.status_label.config(text="좌표 불러오기 실패")

    def start_clicking(self):
        if self.running:
            return
        self.running = True
        self.status_label.config(text="실행 중...")
        self.start_btn.config(state=tk.DISABLED)
        self.stop_btn.config(state=tk.NORMAL)

        coords = []
        try:
            for x_entry, y_entry in self.entries:
                x = int(x_entry.get())
                y = int(y_entry.get())
                coords.append((x, y))
        except ValueError:
            self.status_label.config(text="좌표를 올바르게 입력하세요.")
            self.running = False
            self.start_btn.config(state=tk.NORMAL)
            self.stop_btn.config(state=tk.DISABLED)
            return

        def click_once():
            for x, y in coords:
                if not self.running:
                    break
                pyautogui.click(x, y)
                time.sleep(2)  # 고정 2초 간격
            self.running = False
            self.status_label.config(text="1회 클릭 완료 - 대기 중")
            self.start_btn.config(state=tk.NORMAL)
            self.stop_btn.config(state=tk.DISABLED)

        threading.Thread(target=click_once, daemon=True).start()

    def stop_clicking(self):
        self.running = False
        self.status_label.config(text="정지됨")
        self.start_btn.config(state=tk.NORMAL)
        self.stop_btn.config(state=tk.DISABLED)

if __name__ == "__main__":
    root = tk.Tk()
    app = AutoClickerApp(root)
    root.mainloop()
