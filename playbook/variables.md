thứ tự overide các biến trong Ansible:
1. Command-line values (for example, -u my_user, these are not variables)
2. Role defaults (as defined in Role directory structure) 1
3. Inventory file or script group vars 2
4. Inventory group_vars/all 3
5. Playbook group_vars/all 3
6. Inventory group_vars/* 3
7. Playbook group_vars/* 3
8. Inventory file or script host vars 2
9. Inventory host_vars/* 3
10. Playbook host_vars/* 3
11. Host facts and cached set_facts 4
12. Play vars
13. Play vars_prompt
14. Play vars_files
15. Role vars (as defined in Role directory structure)
16. Block vars (for tasks in block only)
17. Task vars (for the task only)
18. include_vars
19. Registered vars and set_facts
20. Role (and include_role) params
21. include params
22. Extra vars (for example, -e "user=my_user")(always win precedence)

sử dụng  keyword environment ở cấp độ play, block hoặc task để thiết lập biến môi trường cho các hành động trên remote host. Với keyword này, bạn có thể kích hoạt proxy cho các task thực hiện HTTP requests, thiết lập các biến môi trường cần thiết cho các trình quản lý phiên bản theo ngôn ngữ lập trình, và nhiều hơn nữa.
Phạm vi hoạt động:
Khi bạn đặt giá trị với environment: ở cấp độ play hoặc block, nó chỉ khả dụng cho các task bên trong play hoặc block đó và được thực thi bởi cùng một user. Keyword environment: không ảnh hưởng đến bản thân Ansible, các cài đặt cấu hình Ansible, môi trường của các user khác, hoặc việc thực thi các plugin khác như lookups và filters.
Lưu ý quan trọng:
Các biến được đặt bằng environment: không tự động trở thành Ansible facts, ngay cả khi bạn đặt chúng ở cấp độ play. Bạn phải bao gồm một task gather_facts rõ ràng trong playbook và đặt keyword environment trên task đó để biến những giá trị này thành Ansible facts. 
Ví du:
- sử dụng ở task level:
```ansible
- hosts: all
  remote_user: root
  tasks:
    - name: Install cobbler
      ansible.builtin.package:
        name: cobbler
        state: present
      environment:
        http_proxy: http://proxy.example.com:8080
```
- Bạn có thể tái sử dụng các cài đặt environment bằng cách định nghĩa chúng như các biến (variables) trong play của bạn và truy cập chúng trong task giống như cách bạn truy cập bất kỳ biến Ansible đã lưu trữ nào khác.
```
- hosts: all
  remote_user: root

  # create a variable named "proxy_env" that is a dictionary
  vars:
    proxy_env:
      http_proxy: http://proxy.example.com:8080

  tasks:

    - name: Install cobbler
      ansible.builtin.package:
        name: cobbler
        state: present
      environment: "{{ proxy_env }}"
```
- Bạn có thể lưu trữ các cài đặt environment để tái sử dụng trong nhiều playbook bằng cách định nghĩa chúng trong file group_vars.
```
---
# file: group_vars/boston

ntp_server: ntp.bos.example.com
backup: bak.bos.example.com
proxy_env:
  http_proxy: http://proxy.bos.example.com:8080
  https_proxy: http://proxy.bos.example.com:8080
```
- Bạn có thể thiết lập remote environment ở cấp độ play.
```
- hosts: testing

  roles:
     - php
     - nginx

  environment:
    http_proxy: http://proxy.example.com:8080
```
